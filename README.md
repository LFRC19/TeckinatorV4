
### 1. Crear la Carpeta del Servidor Node.js

```bash
mkdir node_server
cd node_server
```

### 2. Inicializar el Proyecto de Node.js

```bash
npm init -y
```

### 3. Instalar las Dependencias Necesarias

Ejecuta los siguientes comandos para instalar todas las dependencias:

```bash
npm install aki-api express cors google-translate-api-x
```

#### Descripción de las Dependencias

- **`aki-api`**: Biblioteca para interactuar con la API de Akinator y obtener las preguntas.
- **`express`**: Framework para crear el servidor web en Node.js.
- **`cors`**: Middleware que permite manejar políticas CORS y aceptar solicitudes desde otros orígenes, como el frontend en PHP.
- **`google-translate-api-x`**: Biblioteca para traducir las preguntas de Akinator al español usando la API de Google Translate.

### 4. Crear el Archivo `server.js`

Dentro de la carpeta `node_server`, crea un archivo `server.js` con el siguiente contenido:

```javascript
const express = require('express');
const { Aki } = require('aki-api');
const cors = require('cors');
const translate = require('google-translate-api-x');

const app = express();
const PORT = 5000;

app.use(cors());
app.use(express.json());

let akiInstance = null;

app.post('/start', async (req, res) => {
    try {
        akiInstance = new Aki({ region: 'en' });
        await akiInstance.start();

        const translatedQuestion = await translate(akiInstance.question, { from: 'en', to: 'es' });
        res.json({ question: translatedQuestion.text });
    } catch (error) {
        console.error("Error al iniciar el juego:", error);
        res.status(500).json({ error: "No se pudo iniciar el juego de Akinator" });
    }
});

app.post('/answer', async (req, res) => {
    try {
        if (!akiInstance) {
            return res.status(400).json({ error: "Juego no iniciado" });
        }

        const answer = req.body.answer;
        await akiInstance.step(answer);

        if (akiInstance.progress >= 95) {
            await akiInstance.win();
            res.json({
                game_over: true,
                result: {
                    name: akiInstance.answers[0].name,
                    description: akiInstance.answers[0].description,
                    image: akiInstance.answers[0].absolute_picture_path
                }
            });
        } else {
            const translatedQuestion = await translate(akiInstance.question, { from: 'en', to: 'es' });
            res.json({ question: translatedQuestion.text });
        }
    } catch (error) {
        console.error("Error al procesar la respuesta:", error);
        res.status(500).json({ error: "Error al procesar la respuesta" });
    }
});

app.listen(PORT, () => {
    console.log(`Servidor de Akinator ejecutándose en http://localhost:${PORT}`);
});
```

### 5. Ejecutar el Servidor

Para iniciar el servidor, usa el siguiente comando:

```bash
node server.js
```

Con estos pasos, tendrás un servidor funcional en Node.js que interactúa con Akinator y traduce las preguntas al español usando Google Translate.
