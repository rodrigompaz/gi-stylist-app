// Importar la librería de Google Generative AI
const {
  GoogleGenerativeAI,
  HarmCategory,
  HarmBlockThreshold,
} = require("@google/generative-ai");

// Esta es tu función "Serverless" de Vercel.
// Se ejecutará de forma segura en el servidor.
export default async function handler(request, response) {
  // 1. Solo aceptar peticiones POST
  if (request.method !== 'POST') {
    return response.status(405).json({ error: 'Method Not Allowed' });
  }

  // 2. Leer la clave de API desde las variables de entorno de Vercel
  const apiKey = process.env.GEMINI_API_KEY;
  if (!apiKey) {
    return response.status(500).json({ error: 'API key not configured' });
  }

  // 3. Obtener los prompts del cuerpo de la petición (enviados desde el HTML)
  const { systemPrompt, userQuery } = request.body;
  if (!systemPrompt || !userQuery) {
    return response.status(400).json({ error: 'Missing systemPrompt or userQuery' });
  }

  try {
    // 4. Inicializar el modelo Gemini
    const genAI = new GoogleGenerativeAI(apiKey);
    const model = genAI.getGenerativeModel({
      model: "gemini-2.5-flash-preview-09-2025",
      // Activar Google Search
      tools: [
        { "google_search": {} }
      ],
      systemInstruction: systemPrompt,
    });
    
    // 5. Enviar el prompt a Gemini
    const result = await model.generateContent(userQuery);
    const geminiResponse = result.response;

    // 6. Extraer el texto y las fuentes de la respuesta
    const text = geminiResponse.candidates[0].content.parts[0].text;
    
    let sources = [];
    const groundingMetadata = geminiResponse.candidates[0].groundingMetadata;
    
    if (groundingMetadata && groundingMetadata.groundingAttributions) {
        sources = groundingMetadata.groundingAttributions
            .map(attribution => ({
                uri: attribution.web?.uri,
                title: attribution.web?.title,
            }))
            .filter(source => source.uri && source.title); // Filtrar fuentes válidas
    }

    // 7. Enviar la respuesta de vuelta al frontend (HTML)
    return response.status(200).json({
      text: text,
      sources: sources
    });

  } catch (error) {
    console.error("Error en la función de Vercel:", error);
    return response.status(500).json({ error: 'Error al generar contenido: ' + error.message });
  }
}
