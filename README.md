-const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const Task = require('./models/Task');

const app = express();
const PORT = 5000;

app.use(cors());
app.use(express.json());

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/todolist', {
    useNewUrlParser: true,
    useUnifiedTopology: true
});

// Routes
app.get('/tasks', async (req, res) => {
    const tasks = await Task.find();
    res.json(tasks);
});

app.post('/tasks', async (req, res) => {
    const task = new Task(req.body);
    await task.save();
    res.json(task);
});

app.put('/tasks/:id', async (req, res) => {
    const task = await Task.findByIdAndUpdate(req.params.id, req.body, { new: true });
    res.json(task);
});

app.delete('/tasks/:id', async (req, res) => {
    await Task.findByIdAndDelete(req.params.id);
    res.json({ message: 'Task deleted' });
});

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
<!DOCTYPE html>
<html>
<head>
    <title>To-Do List</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        .task { margin-bottom: 10px; }
        .completed { text-decoration: line-through; color: gray; }
    </style>
</head>
<body>
    <h2>To-Do List</h2>
    <input id="newTask" placeholder="Enter task" />
    <button onclick="addTask()">Add Task</button>
    <ul id="taskList"></ul>

    <script>
        const API = 'http://localhost:5000/tasks';

        async function loadTasks() {
            const res = await fetch(API);
            const tasks = await res.json();
            const list = document.getElementById('taskList');
            list.innerHTML = '';
            tasks.forEach(task => {
                const li = document.createElement('li');
                li.className = 'task';
                li.innerHTML = `
                    <span class="${task.completed ? 'completed' : ''}">${task.title}</span>
                    <button onclick="toggleComplete('${task._id}', ${!task.completed})">✓</button>
                    <button onclick="deleteTask('${task._id}')">🗑️</button>
                `;
                list.appendChild(li);
            });
        }

        async function addTask() {
            const input = document.getElementById('newTask');
            const title = input.value.trim();
            if (!title) return;
            await fetch(API, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ title })
            });
            input.value = '';
            loadTasks();
        }

        async function toggleComplete(id, completed) {
            await fetch(`${API}/${id}`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ completed })
            });
            loadTasks();
        }

        async function deleteTask(id) {
            await fetch(`${API}/${id}`, { method: 'DELETE' });
            loadTasks();
        }

        loadTasks();
    </script>
</body>
</html>
const mongoose = require('mongoose');

const TaskSchema = new mongoose.Schema({
    title: String,
    completed: {
        type: Boolean,
        default: false
    }
});

module.exports = mongoose.model('Task', TaskSchema);
