# Financial- Absolutely, let's craft something innovative and useful! How about an AI-Powered Personal Finance Manager? This tool can help users manage their finances, track expenses, and provide financial insights and recommendations using machine learning.

Backend: Node.js & MongoDB
Set Up Project:

shell
mkdir finance-manager
cd finance-manager
npm init -y
npm install express mongoose bcryptjs jsonwebtoken
Create Server (server.js):

javascript
const express = require('express');
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const app = express();
const PORT = process.env.PORT || 3000;

mongoose.connect('mongodb://localhost:27017/finance-manager', { useNewUrlParser: true, useUnifiedTopology: true });

const userSchema = new mongoose.Schema({
    username: String,
    password: String,
    transactions: [{ date: Date, amount: Number, category: String }]
});

const User = mongoose.model('User', userSchema);

app.use(express.json());

app.post('/register', async (req, res) => {
    const { username, password } = req.body;
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = new User({ username, password: hashedPassword });
    await user.save();
    res.send({ message: 'User registered successfully' });
});

app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    const user = await User.findOne({ username });
    if (!user || !await bcrypt.compare(password, user.password)) {
        return res.status(400).send({ message: 'Invalid credentials' });
    }
    const token = jwt.sign({ id: user._id }, 'secretKey');
    res.send({ token });
});

app.get('/transactions', async (req, res) => {
    const token = req.headers['authorization'];
    const decoded = jwt.verify(token, 'secretKey');
    const user = await User.findById(decoded.id);
    res.send(user.transactions);
});

app.post('/transactions', async (req, res) => {
    const token = req.headers['authorization'];
    const decoded = jwt.verify(token, 'secretKey');
    const user = await User.findById(decoded.id);
    user.transactions.push(req.body);
    await user.save();
    res.send(user.transactions);
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
Frontend: React
Set Up Project:

shell
npx create-react-app finance-manager-frontend
cd finance-manager-frontend
npm install axios
Create Components: App.js:

javascript
import React from 'react';
import { BrowserRouter as Router, Route, Routes, Link } from 'react-router-dom';
import Register from './components/Register';
import Login from './components/Login';
import Transactions from './components/Transactions';

const App = () => {
    return (
        <Router>
            <nav>
                <Link to="/register">Register</Link>
                <Link to="/login">Login</Link>
                <Link to="/transactions">Transactions</Link>
            </nav>
            <Routes>
                <Route path="/register" element={<Register />} />
                <Route path="/login" element={<Login />} />
                <Route path="/transactions" element={<Transactions />} />
            </Routes>
        </Router>
    );
};

export default App;
Register.js:

javascript
import React, { useState } from 'react';
import axios from 'axios';

const Register = () => {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();
        await axios.post('http://localhost:3000/register', { username, password });
        setUsername('');
        setPassword('');
    };

    return (
        <form onSubmit={handleSubmit}>
            <input 
                type="text" 
                value={username} 
                onChange={(e) => setUsername(e.target.value)} 
                placeholder="Username" 
            />
            <input 
                type="password" 
                value={password} 
                onChange={(e) => setPassword(e.target.value)} 
                placeholder="Password" 
            />
            <button type="submit">Register</button>
        </form>
    );
};

export default Register;
Login.js:

javascript
import React, { useState } from 'react';
import axios from 'axios';

const Login = () => {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();
        const response = await axios.post('http://localhost:3000/login', { username, password });
        localStorage.setItem('token', response.data.token);
        setUsername('');
        setPassword('');
    };

    return (
        <form onSubmit={handleSubmit}>
            <input 
                type="text" 
                value={username} 
                onChange={(e) => setUsername(e.target.value)} 
                placeholder="Username" 
            />
            <input 
                type="password" 
                value={password} 
                onChange={(e) => setPassword(e.target.value)} 
                placeholder="Password" 
            />
            <button type="submit">Login</button>
        </form>
    );
};

export default Login;
Transactions.js:

javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const Transactions = () => {
    const [transactions, setTransactions] = useState([]);
    const [amount, setAmount] = useState('');
    const [category, setCategory] = useState('');

    useEffect(() => {
        const fetchTransactions = async () => {
            const token = localStorage.getItem('token');
            const response = await axios.get('http://localhost:3000/transactions', {
                headers: { 'Authorization': token }
            });
            setTransactions(response.data);
        };
        fetchTransactions();
    }, []);

    const handleSubmit = async (e) => {
        e.preventDefault();
        const token = localStorage.getItem('token');
        const response = await axios.post('http://localhost:3000/transactions', { amount, category }, {
            headers: { 'Authorization': token }
        });
        setTransactions(response.data);
        setAmount('');
        setCategory('');
    };

    return (
        <div>
            <form onSubmit={handleSubmit}>
                <input 
                    type="number" 
                    value={amount} 
                    onChange={(e) => setAmount(e.target.value)} 
                    placeholder="Amount" 
                />
                <input 
                    type="text" 
                    value={category} 
                    onChange={(e) => setCategory(e.target.value)} 
                    placeholder="Category" 
                />
                <button type="submit">Add Transaction</button>
            </form>
            <ul>
                {transactions.map(transaction => (
                    <li key={transaction._id}>
                        {transaction.category} - ${transaction.amount}
                    </li>
                ))}
            </ul>
        </div>
    );
};

export default Transactions;
Project Description
This project is an AI-Powered Personal Finance Manager that helps users manage their finances. It allows users to register and log in, track their transactions, and get insights into their spending patterns
