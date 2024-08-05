<script>
    import { onMount, onDestroy } from 'svelte';
    import io from 'socket.io-client'; // Import Socket.IO client

    let chart;
    let data;
    let socket;
    let options;
    let historicalData = []; // Array to store temperature data

    onMount(() => {
        // Load Google Charts
        google.charts.load('current', { packages: ['corechart', 'line'] });
        google.charts.setOnLoadCallback(initChart);

        // Initialize Socket.IO connection
        setupSocket();

        function initChart() {
            console.log('Initializing chart...');
            data = new google.visualization.DataTable();
            data.addColumn('datetime', 'Time'); // Use 'datetime' for time-based data
            data.addColumn('number', 'Temperature');

            options = {
                hAxis: {
                    title: 'Time',
                    format: 'HH:mm:ss', // Format the time on the X-axis
                },
                vAxis: {
                    title: 'Temperature (°C)',
                    viewWindow: {
                        min: 0,
                        max: 50
                    }
                },
                curveType: 'function',
                legend: { position: 'bottom' },
                pointsVisible: true, // Ensure points are visible
                pointSize: 5 // Set the size of the points
            };

            chart = new google.visualization.LineChart(document.getElementById('chart_div'));
            chart.draw(data, options);
            console.log('Chart initialized.');
        }

        function setupSocket() {
            // Connect to the WebSocket server
            console.log('Setting up socket connection...');
            socket = io('http://34.47.150.239:3000');

            socket.on('connect', () => {
                console.log('Connected to WebSocket server');
            });

            socket.on('data', (tempData) => {
                console.log('Received temperature data:', tempData);
                addDataPoint(new Date(), parseFloat(tempData.temp));
            });

            socket.on('disconnect', () => {
                console.log('Disconnected from WebSocket server');
            });

            socket.on('error', (error) => {
                console.error('WebSocket error:', error);
            });
        }

        function addDataPoint(time, temperature) {
            console.log('Adding data point at time:', time, 'with temperature:', temperature);

            // Update the historical data array
            historicalData.push([time, temperature]);

            // Clear existing rows and add updated rows
            data.removeRows(0, data.getNumberOfRows());
            historicalData.forEach(point => {
                data.addRow([point[0], point[1]]);
            });

            // Redraw the chart
            chart.draw(data, options);
            console.log('Chart updated with new data point.');
        }

        onDestroy(() => {
            if (socket) {
                socket.disconnect();
            }
        });
    });
</script>

<style>
    #chart_div {
        height: 500px;
        width: 100%;
    }
</style>

<div id="chart_div"></div>
