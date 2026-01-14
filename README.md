<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Empire Habit Tracker</title>
    <style>
        body { font-family: Arial, sans-serif; background: #111; color: #fff; margin: 0; padding: 20px; }
        h1 { text-align: center; color: #ff4500; }
        #auth { text-align: center; margin-bottom: 20px; }
        #tracker { display: none; }
        .habit { background: #222; padding: 10px; margin: 10px 0; border-radius: 5px; display: flex; justify-content: space-between; align-items: center; }
        .habit button { background: #ff4500; border: none; color: #fff; padding: 5px 10px; cursor: pointer; }
        .habit button:hover { background: #cc3700; }
        #chart { margin-top: 20px; }
        canvas { background: #333; border-radius: 5px; }
        #export { margin-top: 20px; text-align: center; }
    </style>
</head>
<body>
    <div id="auth">
        <h1>Login to Your Empire</h1>
        <input type="text" id="username" placeholder="Username" required>
        <input type="password" id="password" placeholder="Password" required>
        <button onclick="login()">Enter</button>
        <p id="auth-msg"></p>
    </div>
    <div id="tracker">
        <h1>Empire Habit Tracker</h1>
        <input type="text" id="habitName" placeholder="New Habit (e.g., Gym)">
        <button onclick="addHabit()">Add Habit</button>
        <div id="habitsList"></div>
        <div id="chart">
            <canvas id="progressChart" width="400" height="200"></canvas>
        </div>
        <div id="export">
            <button onclick="exportData()">Export Data (CSV)</button>
            <button onclick="logout()">Logout</button>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script>
        let habits = [];
        let currentUser = null;
        const users = JSON.parse(localStorage.getItem('users')) || {}; // Simulated auth storage

        function login() {
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;
            if (!users[username]) {
                users[username] = { password, habits: [] };
                localStorage.setItem('users', JSON.stringify(users));
            } else if (users[username].password !== password) {
                document.getElementById('auth-msg').textContent = 'Wrong password. Try again.';
                return;
            }
            currentUser = username;
            habits = users[username].habits;
            document.getElementById('auth').style.display = 'none';
            document.getElementById('tracker').style.display = 'block';
            loadHabits();
            setupReminders();
            updateChart();
        }

        function logout() {
            saveHabits();
            currentUser = null;
            document.getElementById('tracker').style.display = 'none';
            document.getElementById('auth').style.display = 'block';
            document.getElementById('auth-msg').textContent = '';
        }

        function addHabit() {
            const name = document.getElementById('habitName').value;
            if (name) {
                habits.push({ name, streak: 0, lastCompleted: null, history: [] });
                saveHabits();
                loadHabits();
                document.getElementById('habitName').value = '';
            }
        }

        function completeHabit(index) {
            const today = new Date().toDateString();
            if (habits[index].lastCompleted !== today) {
                habits[index].streak++;
                habits[index].lastCompleted = today;
                habits[index].history.push(today);
                saveHabits();
                loadHabits();
                updateChart();
                notify('Habit completed: ' + habits[index].name + '. Streak: ' + habits[index].streak);
            }
        }

        function resetStreaks() {
            const today = new Date().toDateString();
            habits.forEach(h => {
                if (h.lastCompleted !== today) h.streak = 0;
            });
            saveHabits();
            loadHabits();
        }

        function loadHabits() {
            resetStreaks(); // Daily reset
            const list = document.getElementById('habitsList');
            list.innerHTML = '';
            habits.forEach((h, i) => {
                const div = document.createElement('div');
                div.className = 'habit';
                div.innerHTML = `${h.name} - Streak: \( {h.streak} <button onclick="completeHabit( \){i})">Complete</button>`;
                list.appendChild(div);
            });
        }

        function saveHabits() {
            users[currentUser].habits = habits;
            localStorage.setItem('users', JSON.stringify(users));
        }

        function updateChart() {
            const ctx = document.getElementById('progressChart').getContext('2d');
            const labels = habits.map(h => h.name);
            const data = habits.map(h => h.streak);
            new Chart(ctx, {
                type: 'bar',
                data: { labels, datasets: [{ label: 'Streaks', data, backgroundColor: '#ff4500' }] },
                options: { scales: { y: { beginAtZero: true } } }
            });
        }

        function setupReminders() {
            if (Notification.permission !== 'granted') Notification.requestPermission();
            setInterval(() => {
                habits.forEach(h => {
                    if (h.lastCompleted !== new Date().toDateString()) {
                        notify('Reminder: Complete ' + h.name + ' today. No weakness.');
                    }
                });
            }, 3600000); // Hourly check
        }

        function notify(message) {
            if (Notification.permission === 'granted') new Notification(message);
        }

        function exportData() {
            let csv = 'Habit,Streak,History\n';
            habits.forEach(h => {
                csv += `\( {h.name}, \){h.streak},"${h.history.join(',')}"\n`;
            });
            const blob = new Blob([csv], { type: 'text/csv' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'habits.csv';
            a.click();
        }
    </script>
</body>
</html>
