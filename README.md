<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EV Charging Station</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #f0f0f0;
            margin: 0;
            padding: 0;
        }
        .container {
            max-width: 400px;
            margin: 50px auto;
            padding: 20px;
            background-color: #ffffff;
            border-radius: 10px;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
            text-align: center;
        }
        h1 {
            color: #333;
            margin-bottom: 20px;
        }
        input[type="number"], input[type="text"], input[type="password"] {
            width: 100%;
            padding: 12px;
            margin-bottom: 20px;
            border: 1px solid #ccc;
            border-radius: 5px;
            box-sizing: border-box;
            font-size: 16px;
        }
        button {
            width: 100%;
            padding: 12px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            font-size: 16px;
            cursor: pointer;
            margin-bottom: 10px;
        }
        button:hover {
            background-color: #45a049;
        }
        .menu-icon {
            position: absolute;
            top: 15px;
            right: 15px;
            cursor: pointer;
            font-size: 30px;
            color: #333;
        }
        .side-menu {
            height: 100%;
            width: 250px;
            position: fixed;
            top: 0;
            right: -250px;
            background-color: #333;
            color: white;
            overflow-x: hidden;
            transition: 0.3s;
            padding-top: 60px;
        }
        .side-menu a {
            padding: 10px 15px;
            text-decoration: none;
            font-size: 18px;
            color: #fff;
            display: block;
            transition: 0.3s;
        }
        .side-menu a:hover {
            background-color: #575757;
        }
        .side-menu .close-btn {
            position: absolute;
            top: 10px;
            right: 15px;
            font-size: 30px;
        }
        .content {
            display: none;
        }
        .active {
            display: block;
        }
        .status-message {
            margin-top: 10px;
            color: #ff0000;
        }
    </style>
</head>
<body>
    <div class="menu-icon" onclick="toggleMenu()">&#9776;</div>

    <div class="side-menu" id="sideMenu">
        <a href="javascript:void(0)" class="close-btn" onclick="toggleMenu()">&times;</a>
        <a href="#" onclick="showContent('chargeSection')">EV Charging</a>
        <a href="#" onclick="showContent('wifiSection')">Wi-Fi Connect</a>
        <a href="#stationDetails">Station Details</a>
        <a href="#donate">Donate to Charity</a>
    </div>

    <div id="chargeSection" class="container content active">
        <h1>EV Charging Station</h1>

        <p>Enter your current charge percentage:</p>
        <input type="number" id="currentCharge" min="0" max="100" placeholder="Current charge %">

        <p>Enter the distance to travel (in km):</p>
        <input type="number" id="distanceToTravel" min="1" placeholder="Distance in km">

        <p>Calculated desired charge percentage:</p>
        <input type="number" id="desiredCharge" min="1" max="100" placeholder="Desired charge %" readonly>

        <p>Time required to charge (minutes):</p>
        <input type="number" id="timeRequired" placeholder="Charging time" readonly>

        <button onclick="calculateCharge()">Calculate</button>
        <button onclick="sendChargeRequest()">Submit</button>
    </div>

    <div id="wifiSection" class="container content">
        <h1>Wi-Fi Connect</h1>
        <input type="text" id="ssid" placeholder="Enter SSID">
        <input type="password" id="password" placeholder="Enter Password">
        <button id="findNetworkBtn" onclick="findNetwork()">Find Network</button>
        <button id="connectBtn" onclick="connectToWiFi()" style="display:none;">Connect to ESP8266</button>
        <div class="status-message" id="statusMessage"></div>
    </div>

    <script>
        function toggleMenu() {
            var sideMenu = document.getElementById("sideMenu");
            if (sideMenu.style.right === "0px") {
                sideMenu.style.right = "-250px";
            } else {
                sideMenu.style.right = "0px";
            }
        }

        function showContent(sectionId) {
            var sections = document.getElementsByClassName('content');
            for (var i = 0; i < sections.length; i++) {
                sections[i].classList.remove('active');
            }
            document.getElementById(sectionId).classList.add('active');
            toggleMenu();
        }

        function calculateCharge() {
            var currentCharge = parseFloat(document.getElementById("currentCharge").value);
            var distanceToTravel = parseFloat(document.getElementById("distanceToTravel").value);

            if (isNaN(currentCharge) || isNaN(distanceToTravel) || currentCharge < 0 || currentCharge > 100) {
                alert("Please enter valid values for current charge and distance.");
                return;
            }

            var chargeNeededForDistance = (distanceToTravel / 150) * 100;
            var totalChargeNeeded = chargeNeededForDistance + 10;

            var desiredCharge = Math.min(currentCharge + totalChargeNeeded, 100);
            document.getElementById("desiredCharge").value = desiredCharge.toFixed(2);

            var timeRequired = (desiredCharge - currentCharge) * 1;
            document.getElementById("timeRequired").value = timeRequired.toFixed(2);
        }

        function sendChargeRequest() {
            var desiredCharge = document.getElementById("desiredCharge").value;

            if (desiredCharge === "" || desiredCharge < 1 || desiredCharge > 100) {
                alert("Please calculate the desired charge percentage first.");
                return;
            }

            var xhr = new XMLHttpRequest();
            var url = "http://192.168.4.1/charge";
            xhr.open("GET", url + "?percent=" + desiredCharge, true);
            xhr.send();

            xhr.onreadystatechange = function() {
                if (xhr.readyState === 4 && xhr.status === 200) {
                    alert("Charge request sent successfully!");
                }
            };
        }

        function findNetwork() {
            document.getElementById("statusMessage").textContent = "Searching for available networks...";
            setTimeout(function() {
                document.getElementById("statusMessage").textContent = "ESP8266 found! Ready to connect.";
                document.getElementById("connectBtn").style.display = "block";
            }, 2000); // Simulating network search delay
        }

        function connectToWiFi() {
            var ssid = document.getElementById("ssid").value;
            var password = document.getElementById("password").value;

            if (ssid === "" || password === "") {
                alert("Please enter both SSID and password.");
                return;
            }

            var xhr = new XMLHttpRequest();
            var url = "http://192.168.4.1/connect";
            xhr.open("POST", url, true);
            xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
            xhr.send("ssid=" + encodeURIComponent(ssid) + "&password=" + encodeURIComponent(password));

            xhr.onreadystatechange = function() {
                if (xhr.readyState === 4 && xhr.status === 200) {
                    document.getElementById("statusMessage").textContent = "Connection successful!";
                    setTimeout(function() {
                        showContent('chargeSection');
                    }, 2000);
                } else if (xhr.readyState === 4) {
                    document.getElementById("statusMessage").textContent = "Failed to connect. Please try again.";
                }
            };
        }
    </script>
</body>
</html>
