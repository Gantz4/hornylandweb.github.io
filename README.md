﻿# hornylandweb.github.io
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Gestor de Imágenes para VRChat</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .hidden { display: none; }
        .slot { margin-bottom: 15px; }
        .success-message { color: green; font-weight: bold; margin-top: 5px; display: none; }
    </style>
</head>
<body>
    <h2>Gestor de Imágenes para VRChat</h2>
    <div id="authSection">
        <label>Ingresa tu nombre de usuario:</label><br>
        <input type="text" id="username" placeholder="Tu nombre"><br>
        <button onclick="checkUser()">Verificar</button>
    </div>

    <div id="uploadSection" class="hidden">
        <h3>Slots de Imágenes</h3>
        <div id="slots">
            <!-- Los slots se generarán dinámicamente -->
        </div>
    </div>

    <script>
        const ranksUrl = "https://pastebin.com/raw/f4EQTNYZ"; // URL de los rangos
        const githubToken = "TU_TOKEN_PERSONAL"; // Token de GitHub
        const repoOwner = "gantz4"; // Tu usuario de GitHub
        const repoName = "vrchat-slots"; // Nombre del repositorio
        const numSlots = 10; // 10 slots predefinidos
        let authorizedUsers = [];

        // Cargar lista de usuarios autorizados
        async function loadRanks() {
            const response = await fetch(ranksUrl);
            const text = await response.text();
            authorizedUsers = text.split("\n").map(line => line.trim()).filter(line => line);
        }

        // Verificar si el usuario está autorizado
        function checkUser() {
            const username = document.getElementById("username").value.trim();
            if (!username) {
                alert("Por favor, ingresa un nombre de usuario.");
                return;
            }

            loadRanks().then(() => {
                if (authorizedUsers.includes(username)) {
                    document.getElementById("authSection").classList.add("hidden");
                    document.getElementById("uploadSection").classList.remove("hidden");
                    generateSlots();
                } else {
                    alert("No estás autorizado para subir imágenes.");
                }
            });
        }

        // Generar los 10 slots predefinidos
        function generateSlots() {
            const slotsDiv = document.getElementById("slots");
            for (let i = 1; i <= numSlots; i++) {
                const slotDiv = document.createElement("div");
                slotDiv.className = "slot";
                slotDiv.innerHTML = `
                    <label>Slot ${i} (slot${i}.jpg):</label><br>
                    <input type="text" id="slot${i}" placeholder="https://ejemplo.com/imagen${i}.jpg">
                    <button onclick="uploadImage(${i})">Cargar</button>
                    <div id="success${i}" class="success-message">¡Cargada con éxito!</div>
                `;
                slotsDiv.appendChild(slotDiv);
            }
        }

        // Subir la imagen al slot correspondiente
        async function uploadImage(slotNumber) {
            const imageUrl = document.getElementById(`slot${slotNumber}`).value.trim();
            if (!imageUrl) {
                alert(`Por favor, ingresa un enlace válido para el Slot ${slotNumber}.`);
                return;
            }

            const filePath = `slot${slotNumber}.jpg`;

            // Obtener el SHA del archivo actual (si existe)
            const getFileResponse = await fetch(
                `https://api.github.com/repos/${repoOwner}/${repoName}/contents/${filePath}`,
                { headers: { Authorization: `token ${githubToken}` } }
            );
            let sha = null;
            if (getFileResponse.ok) {
                const fileData = await getFileResponse.json();
                sha = fileData.sha;
            }

            // Actualizar o crear el archivo con el nuevo enlace
            const content = btoa(imageUrl); // Codificar el enlace en base64
            const updateResponse = await fetch(
                `https://api.github.com/repos/${repoOwner}/${repoName}/contents/${filePath}`,
                {
                    method: "PUT",
                    headers: {
                        Authorization: `token ${githubToken}`,
                        "Content-Type": "application/json"
                    },
                    body: JSON.stringify({
                        message: `Actualizar Slot ${slotNumber}`,
                        content: content,
                        sha: sha // Si no existe, se creará
                    })
                }
            );

            const successMessage = document.getElementById(`success${slotNumber}`);
            if (updateResponse.ok) {
                successMessage.style.display = "block";
                setTimeout(() => {
                    successMessage.style.display = "none";
                }, 3000); // Ocultar después de 3 segundos
            } else {
                alert(`Error al actualizar el Slot ${slotNumber}.`);
            }
        }
    </script>
</body>
</html>
