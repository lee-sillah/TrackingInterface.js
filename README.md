# TrackingInterface.js
To display the status of ideas and to allow users to update the progress of ideas

// frontend/src/components/TrackingInterface.js

import React, { UseEffect, useState } from 'react';
import axios from 'axios';

const TrackingInterface = () => {
    const [ideas, setIdeas] = useState([]);
    const [error, setError] = UseState(null);
    const [selectedIdeaId, setSelectedIdeaId] = UseState('');
    const [status, setStatus] = useState('');

    UseEffect(() => {
        const fetchIdeas = async () => {
            try {
                const response = await Axios.get('/Api/ideas');
                setIdeas(response.data);
            } catch (err) {
                SetError('Failed to load ideas.');
            }
        };
        fetchIdeas();
    }, []);

    const handleStatusUpdate = async (event) => {
        event.preventDefault();
        try {
            await Axios.put(`/Api/ideas/${selectedIdeaId}`, { status });
            setStatus('');
            // Optionally refresh ideas or update local state
        } catch (err) {
            SetError('Failed to update status.');
        }
    };

    return (
        <div>
            <h2>Track and Develop Ideas</h2>
            {error && <p className="error">{error}</p>}
            <h3>Current Ideas</h3>
            <ul>
                {ideas.map((idea) => (
                    <li key={idea._id}>
                        <h4>{idea.title}</h4>
                        <p>Status: {idea.status}</p>
                        <button onClick={() => {
                            setSelectedIdeaId(idea._id);
                            setStatus(idea.status);
                        }}>Update Status</button>
                    </li>
                ))}
            </ul>
            {selectedIdeaId && (
                <form onSubmit={handleStatusUpdate}>
                    <label>
                        New Status:
                        <input 
                            type="text" 
                            value={status} 
                            onChange={(e) => setStatus(e.target.value)} 
                            required 
                        />
                    </label>
                    <button type="submit">Submit Update</button>
                </form>
            )}
        </div>
    );
};

export default TrackingInterface;

// backend/src/models/Idea.js

const mongoose = require('mongoose');

const IdeaSchema = new mongoose.Schema({
    title: { type: String, required: true },
    description: { type: String, required: true },
    category: { type: String, required: false },
    SubmittedAt: { type: Date, default: Date.now },
    votes: { type: Number, default: 0 },
    status: { type: String, default: 'Pending' }, // New field for status
});

module.exports = mongoose.model('Idea', IdeaSchema);

// backend/src/controllers/ideaController.js

const Idea = require('../models/Idea');

exports.updateIdeaStatus = async (req, res) => {
    const { status } = req.body;
    try {
        const idea = await Idea.findByIdAndUpdate(req.params.id, { status }, { new: true });
        if (!idea) {
            return res.status(404).json({ error: 'Idea not found.' });
        }
        res.status(200).json(idea);
    } catch (error) {
        res.status(400).json({ error: 'Failed to update status.' });
    }
};

// backend/src/routes/ideaRoutes.js

const express = require('express');
const router = express.Router();
const ideaController = require('../controllers/ideaController');

// Existing routes...
router.put('/:id', ideaController.updateIdeaStatus); // Update status route

module.exports = router;

// backend/src/server.js

const express = require('express');
const mongoose = require('mongoose');
const ideaRoutes = require('./routes/ideaRoutes');

const app = express();
app.use(express.json()); // for parsing application/json

// MongoDB connection
mongoose.connect('mongodb://localhost/green_future', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
});

// Routes
app.use('/api/ideas', ideaRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
