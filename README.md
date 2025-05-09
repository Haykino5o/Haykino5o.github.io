# Haykino5o.github.io
HAY kino 
<!DOCTYPE html>
<html lang="hy">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IPTV Նվագարկիչ</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #181818;
            color: white;
            margin: 0;
            padding: 0;
            text-align: center;
        }

        header {
            background-color: #333;
            padding: 15px 0;
        }

        header h1 {
            color: white;
            font-size: 36px;
            margin: 0;
        }

        #player {
            max-width: 960px;
            margin: 40px auto;
            background-color: #111;
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.8);
        }

        .video-container {
            position: relative;
            background-color: #000;
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.5);
        }

        video {
            width: 100%;
            height: 100%;
            background-color: #000;
            border-radius: 10px;
        }

        select, input, button {
            margin-top: 20px;
            padding: 12px;
            border-radius: 6px;
            border: 1px solid #444;
            background-color: #333;
            color: white;
            width: 80%;
            font-size: 16px;
        }

        button {
            background-color: #ff0000;
            cursor: pointer;
            width: 80%;
            margin-top: 20px;
        }

        button:hover {
            background-color: #d10000;
        }

        .channel-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: #222;
            margin-bottom: 10px;
            padding: 10px;
            border-radius: 5px;
        }

        .channel-item button {
            background-color: #e74c3c;
            padding: 8px 15px;
            border-radius: 5px;
            border: none;
            color: white;
            font-size: 14px;
        }

        .channel-item button:hover {
            background-color: #c0392b;
        }

        #search-input {
            margin-bottom: 20px;
            padding: 10px;
            width: 80%;
            font-size: 16px;
        }

        #admin-panel {
            display: none;
            background-color: #333;
            padding: 15px;
            border-radius: 10px;
            margin-top: 20px;
        }

        .admin-panel-input {
            width: 100%;
            margin: 10px 0;
            padding: 10px;
            border-radius: 6px;
            background-color: #222;
            color: white;
            border: none;
            font-size: 16px;
        }

        .admin-panel-button {
            background-color: #2980b9;
            width: 100%;
            padding: 12px;
            border-radius: 6px;
            color: white;
            font-size: 16px;
            cursor: pointer;
        }

        .admin-panel-button:hover {
            background-color: #3498db;
        }
    </style>
</head>
<body>

<header>
    <h1>IPTV Նվագարկիչ</h1>
</header>

<div id="player">
    <div class="video-container">
        <video id="video" controls autoplay>
            <source id="video-source" type="application/x-mpegURL">
            Ձեր բրաուզերը չի աջակցում տեսանյութի նվագարկմանը։
        </video>
    </div>

    <select id="channel-select" onchange="changeChannel(this.value)">
        <option disabled selected>Ընտրեք ալիք</option>
    </select>

    <div>
        <input type="text" id="search-input" placeholder="Փնտրել ալիք" oninput="searchChannel()" />
    </div>

    <h3>Բեռնել M3U Հղում</h3>
    <input type="text" id="m3u-url" placeholder="M3U հղում" />
    <button onclick="loadM3U(document.getElementById('m3u-url').value)">Բեռնել</button>

    <h3>Վերբեռնել M3U ֆայլ</h3>
    <input accept=".m3u" id="m3u-file" type="file" />
    <button onclick="uploadM3U()">Վերբեռնել Ֆայլ</button>
</div>

<div id="admin-panel">
    <h3>Ադմինի Գործիքներ</h3>
    <input class="admin-panel-input" type="text" id="ch-name" placeholder="Ալիքի անուն" />
    <input class="admin-panel-input" type="text" id="ch-url" placeholder="Ալիքի URL" />
    <button class="admin-panel-button" onclick="addChannel()">Ավելացնել Ալիք</button>
    
    <h4>Ալիքների ցանկ</h4>
    <ul id="channel-list"></ul>
</div>

<script>
    const adminPassword = "Admin123"; // Տեղադրեք ադմինի գաղտնաբառը
    const channels = [];

    window.onload = function() {
        const saved = localStorage.getItem("iptvChannels");
        if (saved) {
            channels.push(...JSON.parse(saved));
            updateChannelSelect();
            updateChannelList();
        }
    };

    function addChannel() {
        const name = document.getElementById("ch-name").value.trim();
        const url = document.getElementById("ch-url").value.trim();
        if (name && url) {
            channels.push({ name, url });
            saveChannels();
            updateChannelSelect();
            updateChannelList();
            document.getElementById("ch-name").value = "";
            document.getElementById("ch-url").value = "";
        }
    }

    function deleteChannel(index) {
        channels.splice(index, 1);
        saveChannels();
        updateChannelSelect();
        updateChannelList();
    }

    function updateChannelSelect() {
        const select = document.getElementById("channel-select");
        select.innerHTML = '<option disabled selected>Ընտրեք ալիք</option>';
        channels.forEach(channel => {
            const option = document.createElement("option");
            option.value = channel.url;
            option.textContent = channel.name;
            select.appendChild(option);
        });
    }

    function updateChannelList() {
        const list = document.getElementById("channel-list");
        list.innerHTML = '';
        channels.forEach((channel, index) => {
            const li = document.createElement("li");
            li.classList.add("channel-item");
            li.innerHTML = `${channel.name} <button onclick="deleteChannel(${index})">Ջնջել</button>`;
            list.appendChild(li);
        });
    }

    function changeChannel(url) {
        const video = document.getElementById("video");
        const source = document.getElementById("video-source");
        source.src = url;
        video.load();
        video.play();
    }

    function saveChannels() {
        localStorage.setItem("iptvChannels", JSON.stringify(channels));
    }

    function loadM3U(url) {
        fetch(url)
            .then(res => res.text())
            .then(data => {
                const lines = data.split('\n');
                let currentName = '';
                lines.forEach(line => {
                    if (line.startsWith('#EXTINF')) {
                        currentName = line.split(',').pop().trim();
                    } else if (line.startsWith('http')) {
                        addChannel(currentName, line.trim());
                    }
                });
            })
            .catch(err => alert('M3U հղումը չի բեռնվել'));
    }

    function uploadM3U() {
        const fileInput = document.getElementById("m3u-file");
        const file = fileInput.files[0];
        if (!file) return;
        const reader = new FileReader();
        reader.onload = function(e) {
            const data = e.target.result;
            const lines = data.split('\n');
            let currentName = '';
            lines.forEach(line => {
                if (line.startsWith('#EXTINF')) {
                    currentName = line.split(',').pop().trim();
                } else if (line.startsWith('http')) {
                    addChannel(currentName, line.trim());
                }
            });
        };
        reader.readAsText(file);
    }

    function searchChannel() {
        const searchTerm = document.getElementById("search-input").value.toLowerCase();
        const filteredChannels = channels.filter(channel => channel.name.toLowerCase().includes(searchTerm));
        updateChannelList(filteredChannels);
    }

</script>

</body>
</html>
