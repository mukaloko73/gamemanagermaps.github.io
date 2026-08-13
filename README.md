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

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        header {
            text-align: center;
            margin-bottom: 30px;
        }

        h1 {
            color: var(--primary);
            margin-bottom: 5px;
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
            transition: transform 0.2s;
        }

        .map-card:hover {
            transform: translateY(-4px);
        }

        .map-title {
            font-size: 1.2rem;
            font-weight: bold;
            margin: 0 0 5px 0;
        }

        .map-author {
            color: var(--subtext);
            font-size: 0.9rem;
            margin-bottom: 20px;
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
            gap: 6px;
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
        // ==========================================
        // SUAS CONFIGURAÇÕES DO SUPABASE
        // ==========================================
        const SUPABASE_URL = "https://iujervnsbozespphjhcm.supabase.co"; // Ex: https://xyz.supabase.co
        const SUPABASE_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Iml1amVydm5zYm96ZXNwcGhqaGNtIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzQwNTIwNDUsImV4cCI6MjA4OTYyODA0NX0.OUGDyjbUnqpHY48fOz0Ulzs57P2rYvZcEXMf-ivghMo";    // Sua anon key do Supabase

        const container = document.getElementById('mapsContainer');

        // Carrega a lista de mapas ao abrir a página
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

        // Renderiza os cards dos mapas
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

        // Função para curtir o mapa via RPC (igual ao Unity)
        async function likeMap(mapId) {
            const btn = document.getElementById(`btn-${mapId}`);
            const likesSpan = document.getElementById(`likes-${mapId}`);

            // Desabilita o botão para evitar spams
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
                    // Atualiza a contagem na tela localmente
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

        // Previne XSS ao exibir nomes de usuários/mapas
        function escapeHtml(str) {
            return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }

        // Inicializa
        fetchMaps();
    </script>
</body>
</html>
