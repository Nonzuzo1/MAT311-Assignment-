<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Bisection Method Root Finder</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/11.8.0/math.js"></script>
    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
    <style>
        :root {
            --primary-color: #1a5f7a;
            --secondary-color: #2c3e50;
            --tertiary-color: #7f8c8d;
            --background-color: #f5f5f5;
            --text-color: #333;
        }

        body {
            font-family: 'Georgia', serif;
            line-height: 1.6;
            color: var(--text-color);
            background-color: var(--background-color);
            margin: 0;
            padding: 0;
            transition: all 0.3s ease;
        }

        .container {
            display: flex;
            min-height: 100vh;
        }

        .input-page, .output-page {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
        }

        .input-page {
            background-color: white;
            border-right: 1px solid #ddd;
        }

        .output-page {
            background-color: #f9f9f9;
            display: none;
        }

        h1, h2 {
            text-align: center;
            color: var(--primary-color);
            font-family: 'Palatino Linotype', 'Book Antiqua', Palatino, serif;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }

        input[type="text"], input[type="number"], select {
            width: 100%;
            padding: 8px;
            margin-bottom: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-family: 'Georgia', serif;
        }

        button {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 10px 20px;
            cursor: pointer;
            border-radius: 4px;
            transition: background-color 0.3s ease;
            font-family: 'Georgia', serif;
        }

        button:hover {
            background-color: #14495e;
        }

        #result, #ivt-result {
            margin-top: 20px;
            padding: 10px;
            background-color: white;
            border-radius: 4px;
            border: 1px solid #ddd;
        }

        #bisectionGraph, #iterationGraph {
            width: 100%;
            height: 400px;
            margin: 20px auto;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }

        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }

        th {
            background-color: var(--primary-color);
            color: white;
        }

        tr:nth-child(even) {
            background-color: #f2f2f2;
        }

        tr:hover {
            background-color: #ddd;
            cursor: pointer;
        }

        .accessibility-controls {
            margin-bottom: 20px;
        }

        .high-contrast {
            --primary-color: #000;
            --secondary-color: #fff;
            --tertiary-color: #888;
            --background-color: #fff;
            --text-color: #000;
        }

        .example-functions {
            margin-top: 20px;
            padding: 10px;
            background-color: #e6f3ff;
            border-radius: 4px;
        }

        @media (max-width: 768px) {
            .container {
                flex-direction: column;
            }

            .input-page, .output-page {
                height: auto;
            }

            input[type="text"], input[type="number"], select {
                font-size: 16px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="input-page">
            <h1>Advanced Bisection Method Root Finder</h1>
            
            <div class="accessibility-controls">
                <button onclick="toggleHighContrast()">Toggle High Contrast</button>
                <button onclick="increaseFontSize()">Increase Font Size</button>
                <button onclick="decreaseFontSize()">Decrease Font Size</button>
            </div>

            <div class="input-section">
                <label for="function">Function:</label>
                <input type="text" id="function" placeholder="e.g., x^2 - 4">

                <label for="interval">Interval [a, b]:</label>
                <input type="text" id="interval" placeholder="e.g., 0,3">

                <label for="iterations">Number of Iterations:</label>
                <input type="number" id="iterations" min="1" value="10">

                <label for="tolerance">Tolerance (optional):</label>
                <input type="number" id="tolerance" min="0" step="0.000001" placeholder="e.g., 0.0001">

                <button onclick="calculate()">Calculate</button>
            </div>

            <div class="example-functions">
                <h2>Example Functions</h2>
                <select id="exampleSelect" onchange="loadExample()">
                    <option value="">Select an example</option>
                    <option value="polynomial">Polynomial: x^3 - x - 2</option>
                    <option value="rational">Rational: (x^2 - 1) / (x - 2)</option>
                    <option value="exponential">Exponential: exp(x) - 3</option>
                    <option value="logarithmic">Logarithmic: log(x) - 1</option>
                    <option value="trigonometric">Trigonometric: sin(x) - 0.5</option>
                    <option value="radical">Radical: sqrt(x) - 2</option>
                    <option value="nonlinear">Nonlinear: x^2 + sin(x) - 4</option>
                </select>
            </div>
        </div>

        <div class="output-page">
            <h1>Results</h1>
            <button onclick="showInputPage()">Back to Input</button>
            <div id="result"></div>
            <div id="ivt-result"></div>
            <div id="bisectionGraph"></div>
            <div id="iterationTable"></div>
            <div id="iterationGraph"></div>
        </div>
    </div>

    <script>
        let fontSize = 16;
        let funcStr, a, b;

        function toggleHighContrast() {
            document.body.classList.toggle('high-contrast');
        }

        function increaseFontSize() {
            fontSize += 2;
            document.body.style.fontSize = `${fontSize}px`;
        }

        function decreaseFontSize() {
            if (fontSize > 12) {
                fontSize -= 2;
                document.body.style.fontSize = `${fontSize}px`;
            }
        }

        function showOutputPage() {
            document.querySelector('.input-page').style.display = 'none';
            document.querySelector('.output-page').style.display = 'block';
        }

        function showInputPage() {
            document.querySelector('.input-page').style.display = 'block';
            document.querySelector('.output-page').style.display = 'none';
        }

        function loadExample() {
            const select = document.getElementById('exampleSelect');
            const functionInput = document.getElementById('function');
            const intervalInput = document.getElementById('interval');

            switch (select.value) {
                case 'polynomial':
                    functionInput.value = 'x^3 - x - 2';
                    intervalInput.value = '1,2';
                    break;
                case 'rational':
                    functionInput.value = '(x^2 - 1) / (x - 2)';
                    intervalInput.value = '0,1';
                    break;
                case 'exponential':
                    functionInput.value = 'exp(x) - 3';
                    intervalInput.value = '0,2';
                    break;
                case 'logarithmic':
                    functionInput.value = 'log(x) - 1';
                    intervalInput.value = '2,4';
                    break;
                case 'trigonometric':
                    functionInput.value = 'sin(x) - 0.5';
                    intervalInput.value = '0,2';
                    break;
                case 'radical':
                    functionInput.value = 'sqrt(x) - 2';
                    intervalInput.value = '3,5';
                    break;
                case 'nonlinear':
                    functionInput.value = 'x^2 + sin(x) - 4';
                    intervalInput.value = '1,3';
                    break;
            }
        }

        function calculate() {
            funcStr = document.getElementById('function').value;
            const intervalStr = document.getElementById('interval').value;
            const iterations = parseInt(document.getElementById('iterations').value);
            const tolerance = parseFloat(document.getElementById('tolerance').value) || 0;

            [a, b] = intervalStr.split(',').map(Number);

            const func = x => math.evaluate(funcStr, { x: x });

            if (isNaN(a) || isNaN(b)) {
                alert("Please enter valid interval values.");
                return;
            }

            const ivtResult = checkIVT(func, a, b);
            document.getElementById('ivt-result').innerHTML = ivtResult.message;

            if (!ivtResult.hasRoot) {
                document.getElementById('result').innerHTML = "No root exists in the given interval.";
                showOutputPage();
                return;
            }

            const result = bisectionMethod(func, a, b, iterations, tolerance);
            displayResult(result);
            drawBisectionGraph(result.iterations);
            createIterationTable(result.iterations);
            showOutputPage();
        }

        function checkIVT(func, a, b) {
            const fa = func(a);
            const fb = func(b);

            if (fa * fb > 0) {
                return {
                    hasRoot: false,
                    message: `The Intermediate Value Theorem does not guarantee a root in the interval [${a}, ${b}] because f(${a}) and f(${b}) have the same sign.`
                };
            } else {
                return {
                    hasRoot: true,
                    message: `The Intermediate Value Theorem guarantees at least one root in the interval [${a}, ${b}] because f(${a}) and f(${b}) have opposite signs.`
                };
            }
        }

        function bisectionMethod(func, a, b, maxIterations, tolerance) {
            let iterations = [];
            let c, fc;

            for (let i = 0; i < maxIterations; i++) {
                c = (a + b) / 2;
                fc = func(c);

                iterations.push({ a, b, c, fa: func(a), fb: func(b), fc });

                if (Math.abs(fc) < tolerance || (b - a) / 2 < tolerance) {
                    return { root: c, iterations, iterationCount: i + 1 };
                }

                if (func(a) * fc < 0) {
                    b = c;
                } else {
                    a = c;
                }
            }

            return { root: c, iterations, iterationCount: maxIterations };
        }

        function displayResult(result) {
            document.getElementById('result').innerHTML = `
                <p>Approximate root: ${result.root.toFixed(6)}</p>
                <p>Number of iterations: ${result.iterationCount}</p>
            `;
        }

        function drawBisectionGraph(iterations) {
            const trace = {
                x: iterations.map((_, i) => i + 1),
                y: iterations.map(it => it.c),
                mode: 'lines+markers',
                type: 'scatter',
                name: 'Midpoint (c)'
            };

            const layout = {
                title: 'Bisection Method Convergence',
                xaxis: { title: 'Iteration' },
                yaxis: { title: 'Midpoint Value' }
            };

            Plotly.newPlot('bisectionGraph', [trace], layout);
        }

        function createIterationTable(iterations) {
            const table = document.createElement('table');
            table.innerHTML = `
                <tr>
                    <th>Iteration</th>
                    <th>a</th>
                    <th>b</th>
                    <th>c</th>
                    <th>f(a)</th>
                    <th>f(b)</th>
                    <th>f(c)</th>
                </tr>
            `;

            iterations.forEach((iteration, index) => {
                const row = table.insertRow();
                row.innerHTML = `
                    <td>${index + 1}</td>
                    <td>${iteration.a.toFixed(6)}</td>
                    <td>${iteration.b.toFixed(6)}</td>
                    <td>${iteration.c.toFixed(6)}</td>
                    <td>${iteration.fa.toFixed(6)}</td>
                    <td>${iteration.fb.toFixed(6)}</td>
                    <td>${iteration.fc.toFixed(6)}</td>
                `;
                row.onclick = () => drawIterationGraph(iteration, index + 1);
            });

            const tableContainer = document.getElementById('iterationTable');
            tableContainer.innerHTML = '';
            tableContainer.appendChild(table);
        }

        function drawIterationGraph(iteration, iterationNumber) {
            const func = x => math.evaluate(funcStr, { x: x });
            
            const x = math.range(Math.min(a, b), Math.max(a, b), 0.01).toArray();
            const y = x.map(func);

            const trace1 = {
                x: x,
                y: y,
                mode: 'lines',
                name: 'f(x)'
            };

            const trace2 = {
                x: [iteration.a, iteration.b, iteration.c],
                y: [iteration.fa, iteration.fb, iteration.fc],
                mode: 'markers',
                name: 'Bisection points',
                marker: {
                    size: 10,
                    color: ['red', 'blue', 'green']
                }
            };

            const layout = {
                title: `Bisection Method - Iteration ${iterationNumber}`,
                xaxis: { title: 'x' },
                yaxis: { title: 'f(x)' },
                showlegend: true
            };

            Plotly.newPlot('iterationGraph', [trace1, trace2], layout);
        }
    </script>
</body>
</html>
