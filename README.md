<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Licensure Exam for Teachers (LET) Dates and Result Releases</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            padding: 10px;
            border: 1px solid #ccc;
            text-align: center;
        }
        th {
            background-color: #f2f2f2;
        }
        form {
            margin-top: 20px;
        }
        input[type="number"], input[type="date"] {
            padding: 10px;
            margin: 5px;
            width: 200px;
        }
        button {
            padding: 10px 15px;
            margin: 5px;
            cursor: pointer;
        }
        .result {
            margin-top: 20px;
        }
        #chartContainer {
            width: 100%;
            height: 400px;
            margin-top: 20px;
        }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>

<h1>Licensure Exam for Teachers (LET) Dates and Result Releases</h1>

<table id="examTable">
    <thead>
        <tr>
            <th>Year</th>
            <th>Exam Date</th>
            <th>Working Days Until Release</th>
            <th>Number of Takers</th>
            <th>Number of Passers</th>
            <th>Passing Rate (%)</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        <!-- Exam data will be inserted here -->
    </tbody>
</table>

<h2>Add New Exam Entry</h2>
<form id="entryForm">
    <input type="number" id="year" placeholder="Year" required>
    <input type="date" id="examDate" required>
    <input type="number" id="workingDays" placeholder="Working Days Until Release" required>
    <input type="number" id="numTakers" placeholder="Number of Takers" required>
    <input type="number" id="numPassers" placeholder="Number of Passers" required>
    <button type="submit">Add Entry</button>
</form>

<h2>Calculations</h2>
<button onclick="calculateAveragePassingRate()">Calculate Average Passing Rate</button>
<button onclick="predictPassingRate()">Predict Passing Rate for Next Exam</button>
<button onclick="predictDaysUntilNextExam()">Predict Days Until Next Exam Release</button>
<button onclick="clearCalculations()">Clear Calculations</button>

<div class="result" id="resultsDisplay"></div>

<!-- Chart Container -->
<div id="chartContainer">
    <canvas id="passingRateChart"></canvas>
</div>

<script>
    const examTable = document.getElementById('examTable').getElementsByTagName('tbody')[0];
    const passingRateChartCtx = document.getElementById('passingRateChart').getContext('2d');
    const passingRateChart = new Chart(passingRateChartCtx, {
        type: 'line',
        data: {
            labels: [], // Year labels
            datasets: [{
                label: 'Passing Rate (%)',
                data: [], // Passing rate data
                borderColor: 'rgba(75, 192, 192, 1)',
                borderWidth: 2,
                fill: false
            }]
        },
        options: {
            scales: {
                y: {
                    beginAtZero: true,
                    max: 100,
                    title: {
                        display: true,
                        text: 'Passing Rate (%)'
                    }
                },
                x: {
                    title: {
                        display: true,
                        text: 'Exam Year'
                    }
                }
            }
        }
    });

    // Load data from localStorage when the page is loaded
    window.onload = function() {
        loadEntries();
    };

    document.getElementById('entryForm').addEventListener('submit', function (e) {
        e.preventDefault();
        addEntry();
    });

    function addEntry() {
        const year = document.getElementById('year').value;
        const examDate = document.getElementById('examDate').value;
        const workingDays = document.getElementById('workingDays').value;
        const numTakers = document.getElementById('numTakers').value;
        const numPassers = document.getElementById('numPassers').value;

        if (numTakers > 0) {
            const passingRate = ((numPassers / numTakers) * 100).toFixed(2);
            saveEntryToLocalStorage(year, examDate, workingDays, numTakers, numPassers, passingRate);
            loadEntries(); // Reload entries after adding new one to ensure sorting
        } else {
            alert("Number of Takers must be greater than zero.");
        }
    }

    function saveEntryToLocalStorage(year, examDate, workingDays, numTakers, numPassers, passingRate) {
        let entries = JSON.parse(localStorage.getItem('examEntries')) || [];
        entries.push({ year, examDate, workingDays, numTakers, numPassers, passingRate });
        entries.sort((a, b) => a.year - b.year); // Sort entries by year
        localStorage.setItem('examEntries', JSON.stringify(entries));
    }

    function loadEntries() {
        // Clear existing rows in the table and chart
        examTable.innerHTML = '';
        passingRateChart.data.labels = [];
        passingRateChart.data.datasets[0].data = [];

        const entries = JSON.parse(localStorage.getItem('examEntries')) || [];
        entries.sort((a, b) => a.year - b.year); // Sort entries by year before displaying

        entries.forEach(entry => {
            const newRow = examTable.insertRow();
            newRow.innerHTML = `
                <td>${entry.year}</td>
                <td>${entry.examDate}</td>
                <td>${entry.workingDays}</td>
                <td>${entry.numTakers}</td>
                <td>${entry.numPassers}</td>
                <td>${entry.passingRate}%</td>
                <td><button onclick="editEntry(this)">Edit</button> <button onclick="deleteEntry(this)">Delete</button></td>
            `;

            passingRateChart.data.labels.push(entry.year);
            passingRateChart.data.datasets[0].data.push(entry.passingRate);
        });

        passingRateChart.update();
    }

    function editEntry(button) {
        const row = button.parentNode.parentNode;
        const cells = row.getElementsByTagName('td');
        document.getElementById('year').value = cells[0].innerText;
        document.getElementById('examDate').value = cells[1].innerText;
        document.getElementById('workingDays').value = cells[2].innerText;
        document.getElementById('numTakers').value = cells[3].innerText;
        document.getElementById('numPassers').value = cells[4].innerText;
        deleteEntry(button); // Delete row so it can be re-added with updated values
    }

    function deleteEntry(button) {
        const row = button.parentNode.parentNode;
        const cells = row.getElementsByTagName('td');
        const year = cells[0].innerText;
        row.remove();
        removeEntryFromLocalStorage(year); // Remove from localStorage
        loadEntries(); // Reload entries after deleting one to ensure sorting
    }

    function removeEntryFromLocalStorage(year) {
        let entries = JSON.parse(localStorage.getItem('examEntries')) || [];
        entries = entries.filter(entry => entry.year !== year);
        localStorage.setItem('examEntries', JSON.stringify(entries));
    }

    function calculateAveragePassingRate() {
        const entries = JSON.parse(localStorage.getItem('examEntries')) || [];
        const totalRate = entries.reduce((sum, entry) => sum + parseFloat(entry.passingRate), 0);
        const averageRate = (totalRate / entries.length).toFixed(2);
        document.getElementById('resultsDisplay').innerText = `Average Passing Rate: ${averageRate}%`;
    }

    function predictPassingRate() {
        const entries = JSON.parse(localStorage.getItem('examEntries')) || [];
        const totalRate = entries.reduce((sum, entry) => sum + parseFloat(entry.passingRate), 0);
        const predictedRate = (totalRate / entries.length).toFixed(2);
        document.getElementById('resultsDisplay').innerText += `\nPredicted Passing Rate for Next Exam: ${predictedRate}%`;
    }

    function predictDaysUntilNextExam() {
        const entries = JSON.parse(localStorage.getItem('examEntries')) || [];
        const totalDays = entries.reduce((sum, entry) => sum + parseInt(entry.workingDays), 0);
        const predictedDays = (totalDays / entries.length).toFixed(0);
        document.getElementById('resultsDisplay').innerText += `\nPredicted Days Until Next Exam Release: ${predictedDays} days`;
    }

    function clearCalculations() {
        document.getElementById('resultsDisplay').innerText = '';
    }
</script>

</body>
</html>
