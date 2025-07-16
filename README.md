# Programa-Drakuplast
Programa
// === BACKEND: Express.js + PostgreSQL === // Fisier: server.js

const express = require('express'); const cors = require('cors'); const jwt = require('jsonwebtoken'); const bcrypt = require('bcrypt'); const { Pool } = require('pg'); const xlsx = require('xlsx'); const fs = require('fs'); const path = require('path');

const app = express(); const PORT = process.env.PORT || 3001; const SECRET = process.env.JWT_SECRET || 'secrettoken';

app.use(cors()); app.use(express.json());

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// Middleware pentru autentificare function auth(role = null) { return (req, res, next) => { const token = req.headers.authorization?.split(' ')[1]; if (!token) return res.sendStatus(401); jwt.verify(token, SECRET, (err, user) => { if (err) return res.sendStatus(403); if (role && user.role !== role) return res.sendStatus(403); req.user = user; next(); }); }; }

// Login app.post('/api/login', async (req, res) => { const { email, parola } = req.body; const result = await pool.query('SELECT * FROM angajati WHERE email=$1', [email]); if (result.rows.length === 0) return res.status(401).send('Utilizator inexistent');

const user = result.rows[0]; const valid = await bcrypt.compare(parola, user.parola); if (!valid) return res.status(401).send('Parola greșită');

const token = jwt.sign({ id: user.id, role: user.rol }, SECRET); res.json({ token }); });

// Generare automată program app.post('/api/program/genereaza', auth('admin'), async (req, res) => { const { luna, an } = req.body; const result = await pool.query('SELECT id FROM angajati ORDER BY id'); const angajati = result.rows; const zile = new Date(an, luna, 0).getDate();

await pool.query('DELETE FROM program WHERE EXTRACT(MONTH FROM data) = $1 AND EXTRACT(YEAR FROM data) = $2', [luna, an]);

for (let i = 1; i <= zile; i++) { const data = new Date(an, luna - 1, i); const saptamana = Math.floor((i - 1) / 7); angajati.forEach((a, idx) => { const schimb = (idx + saptamana) % 2 === 0 ? 'S1' : 'S2'; pool.query('INSERT INTO program (angajat_id, data, schimb) VALUES ($1, $2, $3)', [a.id, data, schimb]); }); }

res.send('Program generat'); });

// Obținere program pentru o lună app.get('/api/program/:luna/:an', auth(), async (req, res) => { const { luna, an } = req.params; const query =  SELECT a.nume, p.data, p.schimb FROM program p JOIN angajati a ON p.angajat_id = a.id WHERE EXTRACT(MONTH FROM p.data) = $1 AND EXTRACT(YEAR FROM p.data) = $2 ORDER BY a.id, p.data; const result = await pool.query(query, [l

