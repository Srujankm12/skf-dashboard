<script>
    import { onMount, onDestroy } from 'svelte';
    import io from 'socket.io-client'; // Import Socket.IO client

    let chart;
    let data;
    let socket;
    let options;
    let historicalData = {}; // Object to store temperature data by hour

    onMount(() => {
        // Load Google Charts
        google.charts.load('current', { packages: ['corechart', 'line'] });
        google.charts.setOnLoadCallback(initChart);

        // Initialize Socket.IO connection
        setupSocket();

        function initChart() {
            console.log('Initializing chart...');
            data = new google.visualization.DataTable();
            data.addColumn('number', 'Hour');
            data.addColumn('number', 'Temperature');

            // Initialize DataTable with empty values for 24 hours
            for (let i = 0; i < 24; i++) {
                data.addRow([i, null]);
            }

            options = {
                hAxis: {
                    title: 'Hour of the Day',
                    ticks: Array.from({ length: 24 }, (_, i) => ({ v: i, f: i + ':00' }))
                },
                vAxis: {
                    title: 'Temperature (°C)',
                    viewWindow: {
                        min: 0,
                        max: 50
                    }
                },
                curveType: 'function',
                legend: { position: 'bottom' }
            };

            chart = new google.visualization.LineChart(document.getElementById('chart_div'));
            chart.draw(data, options);
            console.log('Chart initialized.');
        }

        function setupSocket() {
            // Connect to the WebSocket server
            console.log('Setting up socket connection...');
            socket = io('http://localhost:3000');

            socket.on('connect', () => {
                console.log('Connected to WebSocket server');
            });

            socket.on('data', (tempData) => {
                const currentHour = new Date().getHours();
                console.log('Received temperature data:', tempData);
                addDataPoint(currentHour, parseFloat(tempData.temp));
            });

            socket.on('disconnect', () => {
                console.log('Disconnected from WebSocket server');
            });

            socket.on('error', (error) => {
                console.error('WebSocket error:', error);
            });
        }

        function addDataPoint(hour, temperature) {
            console.log('Adding data point at hour:', hour, 'with temperature:', temperature);

            // Ensure hour is within valid range
            if (hour < 0 || hour >= 24) {
                console.error('Invalid hour received:', hour);
                return;
            }

            // Update the historical data object
            historicalData[hour] = temperature;

            // Clear existing rows and add updated rows
            data.removeRows(0, data.getNumberOfRows());
            for (let i = 0; i < 24; i++) {
                if (historicalData[i] !== undefined) {
                    data.addRow([i, historicalData[i]]);
                } else {
                    data.addRow([i, null]); // Keep the row for hours without data
                }
            }

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
