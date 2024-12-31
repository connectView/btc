<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BTC/USDT Calculator</title>
    <link href="https://fonts.googleapis.com/css2?family=Komika+Axis&display=swap" rel="stylesheet">
    <style>
        /* General Styles */
        body {
            font-family: 'Komika Axis', sans-serif;
            margin: 0;
            padding: 0;
            color: #f9f9f9;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh; /* Ensure content fills the entire viewport */
            background-size: cover; /* Ensure the image covers the screen */
            background-position: center; /* Center the background image */
            background-repeat: no-repeat;
            transition: background 0.5s ease-in-out; /* Smooth transition when background changes */
        }

        h2 {
            color: #fff;
            text-align: center;
            margin: 20px 0;
            font-size: 2rem; /* Responsive font size */
        }

        /* Table Styles */
        table {
            width: 90%; /* Responsive width */
            max-width: 600px; /* Limit max width */
            border-collapse: separate;
            border-spacing: 15px;
            background: rgba(0, 0, 0, 0.6); /* Transparent black background for readability */
            border-radius: 20px;
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.5);
        }

        th, td {
            padding: 15px;
            text-align: center;
            border-radius: 15px;
            font-size: 1rem; /* Responsive font size */
        }

        th {
            background: rgba(255, 255, 255, 0.2);
        }

        td {
            background: rgba(255, 255, 255, 0.1);
        }

        input {
            width: 100%; /* Full width inside table */
            padding: 10px;
            border: none;
            border-radius: 20px;
            font-size: 1rem; /* Responsive font size */
            background: rgba(255, 255, 255, 0.2);
            color: #fff;
            text-align: center;
        }

        input::placeholder {
            color: #ccc;
        }

        button {
            margin: 20px auto;
            padding: 10px 30px;
            background: linear-gradient(to right, #ff512f, #f09819);
            color: #fff;
            border: none;
            border-radius: 20px;
            cursor: pointer;
            font-size: 1.2rem; /* Responsive font size */
            box-shadow: 0 4px 15px rgba(255, 99, 71, 0.5);
            transition: 0.3s ease;
        }

        button:hover {
            background: linear-gradient(to right, #f09819, #ff512f);
            transform: scale(1.05);
        }

        /* Snowflake Animation */
        .snowflake {
            position: absolute;
            top: -10px;
            font-size: 12px;
            color: rgba(255, 255, 255, 0.6);
            animation: fallSnow 10s linear infinite;
        }

        @keyframes fallSnow {
            0% {
                transform: translateY(0) rotate(0deg);
                opacity: 1;
            }
            100% {
                transform: translateY(100vh) rotate(360deg);
                opacity: 0.5;
            }
        }

        .snowflake.small {
            font-size: 8px;
        }

        .snowflake.large {
            font-size: 18px;
        }
    </style>
</head>
<body>
    <h2>BTC/USDT Live Quantity Calculator</h2>
    <table id="calculationBox">
        <tr>
            <th>Parameter</th>
            <th>Value</th>
        </tr>
        <tr>
            <td>Account Balance (USDT)</td>
            <td><input type="number" id="accountBalance" placeholder="Enter account balance"></td>
        </tr>
        <tr>
            <td>Risk Percentage (%)</td>
            <td><input type="number" id="riskPercentage" placeholder="Enter risk percentage"></td>
        </tr>
        <tr>
            <td>Stop Loss Price (USDT)</td>
            <td><input type="number" id="stopLossPrice" placeholder="Enter stop loss price"></td>
        </tr>
        <tr>
            <td>Live BTC/USDT Price</td>
            <td><span id="livePrice">Connecting...</span></td>
        </tr>
        <tr>
            <td>Risk Per Unit (USDT)</td>
            <td><span id="riskPerUnit">-</span></td>
        </tr>
        <tr>
            <td>Quantity (BTC)</td>
            <td><span id="quantity">-</span></td>
        </tr>
    </table>
    <button onclick="calculate()">Calculate</button>

    <script>
        // Change background based on time
        function setBackground() {
            const currentHour = new Date().getHours(); // Get the current hour
            const body = document.body;

            // Check time range for background selection
            if (currentHour >= 6 && currentHour < 20) {
                // Daytime (6 AM - 8 PM)
                body.style.backgroundImage = "url('https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQBNiItqqf08zh3thZJ_VGfeLjMhZjWQVzgOQ&usqp=CAU')";
            } else {
                // Nighttime (8 PM - 6 AM)
                body.style.backgroundImage = "url('https://images.unsplash.com/photo-1670020111748-9a21ed9fd4ba?fm=jpg&q=60&w=3000&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8NHx8bmlnaHQlMjBtb29ufGVufDB8fDB8fHww')";
            }
        }

        setBackground(); // Set background on page load

        // WebSocket for real-time BTC price updates
        const ws = new WebSocket("wss://stream.binance.com:9443/ws/btcusdt@trade");
        ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            livePrice = parseFloat(data.p);
            document.getElementById("livePrice").innerText = livePrice.toFixed(2);

            calculate();
        };

        ws.onerror = () => {
            document.getElementById("livePrice").innerText = "Error";
        };

        ws.onclose = () => {
            document.getElementById("livePrice").innerText = "Closed";
        };

        // Calculate function
        function calculate() {
            const accountBalance = parseFloat(document.getElementById("accountBalance").value);
            const riskPercentage = parseFloat(document.getElementById("riskPercentage").value);
            const stopLossPrice = parseFloat(document.getElementById("stopLossPrice").value);

            if (!accountBalance || !riskPercentage || !stopLossPrice || !livePrice) return;

            const riskPerUnit = stopLossPrice < livePrice ? livePrice - stopLossPrice : stopLossPrice - livePrice;
            const riskAmount = (accountBalance * riskPercentage) / 100;
            const quantity = riskAmount / riskPerUnit;

            document.getElementById("riskPerUnit").innerText = riskPerUnit.toFixed(2);
            document.getElementById("quantity").innerText = quantity.toFixed(6);
        }
    </script>
</body>
</html>
