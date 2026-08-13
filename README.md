<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Comunidade de Mapas</title>
    <style>
        :root {
            --bg-color: #121214;
            --card-bg: #202024;
            --primary: #8257e5;
            --text: #e1e1e6;
            --subtext: #a8a8b3;
            --like-color: #e25858;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text);
            padding: 40px 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }

        header {
            text-align: center;
            margin-bottom: 40px;
        }

        h1 {
            color: var(--primary);
            font-size: 2.2rem;
            margin-bottom: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
        }

        header p {
            color: var(--subtext);
            font-size: 1rem;
        }

        .maps-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 20px;
            width: 100%;
            max-width: 1000px;
        }

        .map-card {
            background-color: var(--card-bg);
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            border: 1px solid #29292e;
            transition: transform 0.2s, border-color 0.2s;
        }

        .map-card:hover {
            transform: translateY(-4px);
            border-color: var(--primary);
        }

        .map-title {
            font-size: 1.2rem;
            font-weight: bold;
            margin-bottom: 6px;
            word-break: break-word;
        }

        .map-author {
            color: var(--subtext);
            font-size: 0.9rem;
            margin-bottom: 24px;
        }

        .card-footer {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-top: 1px solid #29292e;
            padding-top: 15px;
        }

        .like-btn {
            background: transparent;
            border: 1px solid var(--like-color);
            color: var(--like-color);
            padding: 8px 16px;
            border-radius: 20px;
            cursor: pointer;
            font-weight: bold;
            display: flex;
            align-items: center;
            gap: 8px;
            font-size: 0.95rem;
            transition: all 0.2s;
        }

        .like-btn:hover {
            background-color: var(--like-color);
            color: white;
        }

        .like-btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        .loading, .no-maps {
            text-align: center;
            color: var(--subtext);
            grid-column: 1 / -1;
            font-size: 1.1rem;
            padding: 40px 0;
        }
    </style>
</head>
<body>

    <header>
        <h1>🗺️ Mapas da Comunidade</h1>
        <p>Confira e curta os mapas criados pelos jogadores</p>
    </header>

    <div class="maps-grid" id="mapsContainer">
        <div class="loading">Carregando mapas...</div>
    </div>

    <script>
        const SUPABASE_URL = "https://iujervnsbozespphjhcm.supabase.co"; 
        const SUPABASE_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Iml1amVydm5zYm96ZXNwcGhqaGNtIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzQwNTIwNDUsImV4cCI6MjA4OTYyODA0NX0.OUGDyjbUnqpHY48fOz0Ulzs57P2rYvZcEXMf-ivghMo";    

        const container = document.getElementById('mapsContainer');

        async function fetchMaps() {
            try {
                const response = await fetch(`${SUPABASE_URL}/rest/v1/maps?select=id,map_name,author_name,likes,created_at&order=likes.desc,created_at.desc`, {
                    headers: {
                        'apikey': SUPABASE_KEY,
                        'Authorization': `Bearer ${SUPABASE_KEY}`
                    }
                });

                if (!response.ok) throw new Error('Erro ao buscar mapas');

                const maps = await response.json();
                renderMaps(maps);
            } catch (error) {
                console.error(error);
                container.innerHTML = `<div class="loading">Erro ao carregar mapas. Verifique suas chaves de API.</div>`;
            }
        }

        function renderMaps(maps) {
            container.innerHTML = '';

            if (maps.length === 0) {
                container.innerHTML = `<div class="no-maps">Nenhum mapa encontrado.</div>`;
                return;
            }

            maps.forEach(map => {
                const card = document.createElement('div');
                card.className = 'map-card';
                
                const author = map.author_name ? map.author_name : 'Anônimo';
                const likes = map.likes || 0;

                card.innerHTML = `
                    <div>
                        <div class="map-title">${escapeHtml(map.map_name)}</div>
                        <div class="map-author">por ${escapeHtml(author)}</div>
                    </div>
                    <div class="card-footer">
                        <button class="like-btn" id="btn-${map.id}" onclick="likeMap('${map.id}')">
                            ❤️ <span id="likes-${map.id}">${likes}</span>
                        </button>
                    </div>
                `;

                container.appendChild(card);
            });
        }

        async function likeMap(mapId) {
            const btn = document.getElementById(`btn-${mapId}`);
            const likesSpan = document.getElementById(`likes-${mapId}`);

            btn.disabled = true;

            try {
                const response = await fetch(`${SUPABASE_URL}/rest/v1/rpc/increment_like`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'apikey': SUPABASE_KEY,
                        'Authorization': `Bearer ${SUPABASE_KEY}`
                    },
                    body: JSON.stringify({ map_id: mapId })
                });

                if (response.ok) {
                    let currentLikes = parseInt(likesSpan.textContent) || 0;
                    likesSpan.textContent = currentLikes + 1;
                } else {
                    alert('Não foi possível curtir o mapa.');
                    btn.disabled = false;
                }
            } catch (error) {
                console.error(error);
                btn.disabled = false;
            }
        }

        function escapeHtml(str) {
            return str ? str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;") : '';
        }

        fetchMaps();
    </script>
</body>
</html>
