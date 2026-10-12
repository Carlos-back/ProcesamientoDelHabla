## Conclusiones



**Link del video:** https://drive.google.com/drive/folders/1v9lvb96shN2HQFHLhd-GnKC5Ka\_MHMGa?usp=sharing



### Resumen de lo realizado

Armé un chatbot de preguntas frecuentes sobre **habilitación comercial** con información del Municipio de Bahía Blanca (20 pares pregunta-respuesta). Implementé dos versiones: una con **TF-IDF + similitud del coseno** y otra con **embeddings preentrenados de spaCy (`es\_core\_news\_md`) + similitud del coseno**. Ambas buscan la pregunta del dataset más parecida a la consulta del usuario y devuelven su respuesta, salvo que la similitud no alcance un umbral (0.20 en TF-IDF y 0.40 en embeddings). Las probé con 29 preguntas nuevas: 20 paráfrasis, 3 consultas cortas o informales y 6 preguntas fuera del dataset.

### Resultados

||TF-IDF|Embeddings|
|-|-|-|
|Total (29 pruebas)|17|15|
|Preguntas del tema (23)|12|13|
|Fuera de tema rechazadas (6)|5|2|

### En qué respuestas falla cada chatbot

**TF-IDF** (errores en 11 de las 23 preguntas del tema):

* **Falla cuando la consulta usa sinónimos o palabras que no están en el dataset.** Ejemplos: "Cómo *reservo un horario* para ir a presentar los papeles" (el dataset dice "saco turno") fue respondida con la pregunta de la sociedad; "Es obligatorio tener *extintores* en el local" (el dataset dice "matafuegos") fue respondida con la del plano; y "Cuáles son las *planillas* que firmo" no encontró ninguna palabra en común, por lo que fue rechazada.
* **Se queda con palabras sueltas en lugar de la intención.** "Cómo creo mi usuario en la página del *municipio*" devolvió la respuesta de contacto del Municipio, porque comparte esa palabra.
* **Acepta una pregunta fuera de tema que comparte vocabulario:** "¿Cómo se paga la patente de un auto?" fue respondida con la pregunta de la tasa de habilitación, por la palabra "pagar".

**Embeddings** (errores en 10 de las 23 preguntas del tema):

* **Rechazan muy poco lo que está fuera de tema** (solo 2 de 6). "¿Cuál es la capital de Francia?" fue respondida con la pregunta de cómo iniciar la consulta (similitud 0.52), y "Quiero sacar la licencia de conducir" con la del carnet de manipulador de alimentos (0.59). Los vectores densos casi nunca dan una similitud cercana a cero, por lo que el umbral separa mal lo que corresponde de lo que no.
* **Dan similitudes altas incluso cuando se equivocan** (varios errores entre 0.70 y 0.86), lo que hace difícil usar la similitud como medida de confianza.
* **Confunden preguntas del mismo tema con vocabulario parecido.** Tres de sus errores terminaron en la pregunta "¿Qué tengo que verificar antes de hacer la consulta de habilitación comercial?", que parece actuar como una respuesta "comodín" porque usa palabras muy generales (consulta, habilitación, comercial). Es una hipótesis mía; no la comprobé.

### Qué preguntas confunden

* **Las dos preguntas sobre la Constancia de Cumplimiento Tributario** (validez de 30 días / dónde se pide): los embeddings las confundieron entre sí en ambas direcciones (0.78 y 0.86), porque comparten casi todo el vocabulario y solo cambia una palabra clave. TF-IDF también falló en una de las dos.
* **Las dos preguntas sobre la consulta online** (cómo iniciarla / qué datos cargar): confusión en ambos chatbots.
* **Las preguntas sobre documentación para el turno** (documentos personales, formularios, plano): "Con qué documentos de identidad e impuestos me presento" fue respondida mal por los dos (TF-IDF la confundió con la del turno, y los embeddings con la de la constancia).
* **La pregunta de los matafuegos** con la palabra "extintores": ningún chatbot la resolvió. Es un sinónimo que no está en el dataset, y el modelo de embeddings tampoco logró relacionarlo.

### Comparación de los chatbots

* **TF-IDF** es simple, rápido y no necesita modelos externos. Funciona muy bien cuando la consulta comparte palabras con la pregunta del dataset (en las consultas cortas o casi iguales acertó siempre) y rechaza correctamente lo que no tiene relación. Su límite es que compara palabras, no significados: no reconoce sinónimos y depende de la redacción exacta.
* **Los embeddings** capturan algo de significado, por eso acertaron paráfrasis donde TF-IDF no tenía palabras en común, como "planillas" por "formularios" o "certificado provisorio" por "certificado en trámite". Superaron a TF-IDF en las preguntas del tema (13 contra 12), pero la diferencia es pequeña. Al promediar los vectores de todas las palabras se pierde información, se mezclan preguntas parecidas y casi siempre devuelven alguna respuesta.
* **En el total ganó TF-IDF (17 contra 15)**, pero solo gracias a que rechaza mejor lo que está fuera de tema. Si se mira solo lo que el chatbot debería saber responder, ambos quedaron casi empatados y con bastante margen de error.
* **Conclusión general:** ninguno de los dos es confiable por sí solo en este dataset. Los puntos fuertes son complementarios: TF-IDF sirve para decidir *si* hay una respuesta, y los embeddings para encontrar *cuál* cuando hay sinónimos.

### Limitaciones del trabajo

* El dataset es chico (20 pares) y trata un solo trámite, por lo que muchas preguntas comparten vocabulario y se confunden.
* La lista de prueba tiene solo 29 consultas; una diferencia de uno o dos aciertos no es concluyente.
* Los umbrales (0.20 y 0.40) fueron elegidos a mano. Con los embeddings no encontré un valor que rechazara lo que está fuera de tema sin descartar preguntas válidas.
* Los embeddings de spaCy son estáticos: un solo vector por palabra, sin distinguir polisemia, y las palabras fuera de su vocabulario no tienen vector.

### Posibles mejoras

* **Agregar más formas de preguntar la misma cosa** en el dataset (por ejemplo, 3 o 4 variantes por respuesta, incluyendo sinónimos como "matafuego/extintor" y "turno/horario").
* **Combinar ambos métodos:** usar TF-IDF para decidir si la consulta está dentro del tema y los embeddings para elegir la mejor respuesta.
* Probar un modelo de embeddings más grande (`es\_core\_news\_lg`) o embeddings que manejen mejor las palabras fuera del vocabulario, como FastText, o modelos contextuales como BERT/BETO.

### Fuentes

* Municipio de Bahía Blanca. (s. f.). *Recomendaciones para realizar el trámite* \[Habilitaciones comerciales]. https://www.bahia.gob.ar/tramites/habilitaciones/comerciales/recomendaciones-para-realizar-el-tramite/. Consultado el 11/10/2026.
* Documentación de scikit-learn (`TfidfVectorizer`, `cosine\_similarity`): https://scikit-learn.org
* Documentación de spaCy (modelos en español `es\_core\_news\_sm` y `es\_core\_news\_md`): https://spacy.io/models/es
* Material de clase: vectorización de texto, TF-IDF y word embeddings (clases 1-4 y 7 de Procesamiento del Habla).

