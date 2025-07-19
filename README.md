import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { BarChart, Bar, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts";

interface Team {
  name: string;
  points: number;
  played: number;
  won: number;
  drawn: number;
  lost: number;
  goalsFor: number;
  goalsAgainst: number;
  yellowCards: number;
  redCards: number;
}

interface PlayerStat {
  name: string;
  team: string;
  goals: number;
}

interface Match {
  date: string;
  teamA: string;
  teamB: string;
  scoreA?: number;
  scoreB?: number;
}

const TournamentDashboard = () => {
  const [teams, setTeams] = useState<Team[]>([]);
  const [matches, setMatches] = useState<Match[]>([]);
  const [scorers, setScorers] = useState<PlayerStat[]>([]);

  useEffect(() => {
    const savedTeams = localStorage.getItem("teams");
    const savedMatches = localStorage.getItem("matches");
    const savedScorers = localStorage.getItem("scorers");
    if (savedTeams) setTeams(JSON.parse(savedTeams));
    if (savedMatches) setMatches(JSON.parse(savedMatches));
    if (savedScorers) setScorers(JSON.parse(savedScorers));
  }, []);

  useEffect(() => {
    localStorage.setItem("teams", JSON.stringify(teams));
    localStorage.setItem("matches", JSON.stringify(matches));
    localStorage.setItem("scorers", JSON.stringify(scorers));
  }, [teams, matches, scorers]);

  const addTeam = () => {
    const teamName = prompt("Nombre del equipo:");
    if (teamName) {
      const newTeam: Team = {
        name: teamName,
        points: 0,
        played: 0,
        won: 0,
        drawn: 0,
        lost: 0,
        goalsFor: 0,
        goalsAgainst: 0,
        yellowCards: 0,
        redCards: 0,
      };
      setTeams([...teams, newTeam]);
    }
  };

  const createMatch = () => {
    if (teams.length < 2) return alert("Agrega al menos dos equipos");
    const teamA = prompt("Equipo A:");
    const teamB = prompt("Equipo B:");
    const scoreA = Number(prompt(`Goles de ${teamA}:`));
    const scoreB = Number(prompt(`Goles de ${teamB}:`));
    const yellowA = Number(prompt(`Tarjetas amarillas para ${teamA}:`)) || 0;
    const redA = Number(prompt(`Tarjetas rojas para ${teamA}:`)) || 0;
    const yellowB = Number(prompt(`Tarjetas amarillas para ${teamB}:`)) || 0;
    const redB = Number(prompt(`Tarjetas rojas para ${teamB}:`)) || 0;
    const date = prompt("Fecha del partido (DD/MM/AAAA):") || "Sin fecha";

    if (teamA && teamB && !isNaN(scoreA) && !isNaN(scoreB) && teamA !== teamB) {
      const newMatch: Match = { teamA, teamB, scoreA, scoreB, date };
      setMatches([...matches, newMatch]);
      updateStandings(teamA, teamB, scoreA, scoreB, yellowA, redA, yellowB, redB);
      registerScorers(teamA, scoreA);
      registerScorers(teamB, scoreB);
    }
  };

  const updateStandings = (
    teamAName: string,
    teamBName: string,
    scoreA: number,
    scoreB: number,
    yellowA: number,
    redA: number,
    yellowB: number,
    redB: number
  ) => {
    const updatedTeams = teams.map((team) => {
      if (team.name === teamAName || team.name === teamBName) {
        const isTeamA = team.name === teamAName;
        const goalsFor = isTeamA ? scoreA : scoreB;
        const goalsAgainst = isTeamA ? scoreB : scoreA;
        const yellow = isTeamA ? yellowA : yellowB;
        const red = isTeamA ? redA : redB;
        const won = (isTeamA && scoreA > scoreB) || (!isTeamA && scoreB > scoreA);
        const drawn = scoreA === scoreB;

        return {
          ...team,
          played: team.played + 1,
          goalsFor: team.goalsFor + goalsFor,
          goalsAgainst: team.goalsAgainst + goalsAgainst,
          won: team.won + (won ? 1 : 0),
          drawn: team.drawn + (drawn ? 1 : 0),
          lost: team.lost + (!won && !drawn ? 1 : 0),
          points: team.points + (won ? 3 : drawn ? 1 : 0),
          yellowCards: team.yellowCards + yellow,
          redCards: team.redCards + red,
        };
      }
      return team;
    });
    setTeams(updatedTeams);
  };

  const registerScorers = (teamName: string, goals: number) => {
    for (let i = 0; i < goals; i++) {
      const playerName = prompt(`Nombre del goleador ${i + 1} de ${teamName}`);
      if (playerName) {
        setScorers((prev) => {
          const existing = prev.find(
            (p) => p.name === playerName && p.team === teamName
          );
          if (existing) {
            return prev.map((p) =>
              p.name === playerName && p.team === teamName
                ? { ...p, goals: p.goals + 1 }
                : p
            );
          } else {
            return [...prev, { name: playerName, team: teamName, goals: 1 }];
          }
        });
      }
    }
  };

  const sortedTeams = [...teams].sort((a, b) => b.points - a.points);
  const topScorers = [...scorers].sort((a, b) => b.goals - a.goals);

  const exportToText = () => {
    const text = `TABLA DE POSICIONES:\n\n${sortedTeams
      .map(
        (t, i) =>
          `${i + 1}. ${t.name} - ${t.points} pts, PJ: ${t.played}, G: ${t.won}, E: ${t.drawn}, P: ${t.lost}, GF: ${t.goalsFor}, GC: ${t.goalsAgainst}`
      )
      .join("\n")}\n\nGOLEADORES:\n${topScorers
      .map((p) => `${p.name} (${p.team}) - ${p.goals} goles`)
      .join("\n")}`;

    const blob = new Blob([text], { type: "text/plain;charset=utf-8" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "reporte_torneo.txt";
    a.click();
    URL.revokeObjectURL(url);
  };

  return (
    <div className="p-4 grid gap-4">
      <h1 className="text-2xl font-bold">Gestor de Torneo de Fútbol</h1>

      <Button onClick={exportToText}>📤 Exportar Reporte</Button>

      <Card>
        <CardContent className="p-4">
          <h2 className="font-semibold mb-2">Gráfica de Rendimiento</h2>
          <ResponsiveContainer width="100%" height={200}>
            <BarChart data={teams}>
              <XAxis dataKey="name" />
              <YAxis />
              <Tooltip />
              <Bar dataKey="points" fill="#4ade80" />
            </BarChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>

      <Card>
        <CardContent className="p-4">
          <h2 className="font-semibold mb-2">Calendario de Partidos</h2>
          <ul className="list-disc pl-4">
            {matches.map((match, idx) => (
              <li key={idx}>
                {match.date}: {match.teamA} {match.scoreA} - {match.scoreB} {match.teamB}
              </li>
            ))}
          </ul>
        </CardContent>
      </Card>

      <Card>
        <CardContent className="p-4">
          <h2 className="font-semibold mb-2">Tabla de Posiciones</h2>
          <table className="w-full text-sm">
            <thead>
              <tr>
                <th className="text-left">Equipo</th>
                <th>Pts</th>
                <th>PJ</th>
                <th>G</th>
                <th>E</th>
                <th>P</th>
                <th>GF</th>
                <th>GC</th>
                <th>TA</th>
                <th>TR</th>
              </tr>
            </thead>
            <tbody>
              {sortedTeams.map((team, idx) => (
                <tr key={idx}>
                  <td>{team.name}</td>
                  <td className="text-center">{team.points}</td>
                  <td className="text-center">{team.played}</td>
                  <td className="text-center">{team.won}</td>
                  <td className="text-center">{team.drawn}</td>
                  <td className="text-center">{team.lost}</td>
                  <td className="text-center">{team.goalsFor}</td>
                  <td className="text-center">{team.goalsAgainst}</td>
                  <td className="text-center">{team.yellowCards}</td>
                  <td className="text-center">{team.redCards}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </CardContent>
      </Card>

      <Card>
        <CardContent className="p-4">
          <h2 className="font-semibold mb-2">Goleadores</h2>
          <ul className="list-disc pl-4">
            {topScorers.map((scorer, idx) => (
              <li key={idx}>
                {scorer.name} ({scorer.team}) - {scorer.goals} goles
              </li>
            ))}
          </ul>
        </CardContent>
      </Card>

      <Card>
        <CardContent className="p-4">
          <h2 className="font-semibold mb-2">Equipos Registrados</h2>
          <ul className="list-disc pl-4">
            {teams.map((team, idx) => (
              <li key={idx}>{team.name}</li>
            ))}
          </ul>
          <Button className="mt-2" onClick={addTeam}>
            Agregar Equipo
          </Button>
        </CardContent>
      </Card>

      <Card>
        <CardContent className="p-4">
          <h2 className="font-semibold mb-2">Partidos Programados</h2>
          <Button className="mb-2" onClick={createMatch}>
            Crear Partido
          </Button>
          <ul className="list-disc pl-4">
            {matches.map((match, idx) => (
              <li key={idx}>
                {match.teamA} {match.scoreA} - {match.scoreB} {match.teamB}
              </li>
            ))}
          </ul>
        </CardContent>
      </Card>
    </div>
  );
};

export default TournamentDashboard;
