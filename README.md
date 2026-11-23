# Integrated Agent Project as a Course Assistant

The project integrates a local Python tool, a tool that queries a REST API, calendar tools and completion criteria discovered on an MCP server, short-term memory, a FastAPI API, and optional tracing in LangSmith.

## Requirements

- Python 3.10 or higher
- Valid Gemini API key
- LangSmith account and key only if tracing is enabled

## How to run

1. Create and activate a virtual environment.

2. Install the dependencies.

``bash
pip install -r requirements.txt

``

3. Copy `.env.example` to `.env` and enter the Gemini key.

4. Start the catalog API in a terminal.

```bash
fastapi dev catalogo_api.py --port 8001

``

5. Start the assistant API in another terminal.

``bash
fastapi dev main.py --port 8000

``

6. Access `http://127.0.0.1:8000/docs`.

The MCP server is started by `MCPAdapter` via the stdio transport. It does not need to be started in a third terminal.

## How to enable tracing

Change the variables below in the `.env` file.

``env LANGSMITH_TRACING=true LANGSMITH_API_KEY=your_langsmith_key_here LANGSMITH_PROJECT=pap-cap7-integrated-agent
```

Restart the assistant API. The next executions will be sent to the project specified in LangSmith.

## How to validate

Run the deterministic tests.

``bash
python -m pytest -v

```

With both APIs running, validate the agent and the continuity of the conversation.

``bash
python validate_integration.py

```

## Hands-on completion criteria

The `consultar_criterios_conclusao` tool receives `codigo_curso` and returns the fictitious criteria defined in `dados.py`:

| Course | Minimum grade (0 to 10) | Completed activities (0 to 100%) |

| --- | --- | --- |

| CUR-101 | 7.0 | 75% |

| CUR-202 | 6.0 | 100% |

These data describe course rules. The application does not have individual grades or percentages to state that a student has already met the requirements.

The query normalizes the code, validates the `CUR-` prefix, and returns a copy of the found record. An invalid identifier returns ``{"error": "invalid course code"}`. A code without a record returns ``{"error": "completion criteria not found"}`.

Run the three tests of the new function:

```bash
python -m pytest tests/test_criterios.py -v

```

They check for normalization and success, invalid code, and unregistered criteria, with no model calls or running services. To also check the previous tests, run `python -m pytest -v`.

After changing `mcp_server.py`, restart the main API so that MCP discovery includes the new tool. Keep the catalog running and tracing enabled during integrated validation.

Send the following messages to `POST /chat` using the `conversation_id` and `message` fields. Keep the main API running between shifts.

| conversation_id | message | Expected verification |

| --- | --- | --- |

| academico-hands-on-001 | My registration number is MAT-500. Which course am I in? | Identification of CUR-101 and Python for Automation. |

| academico-hands-on-001 | What are the criteria to complete this course? | Call to the new tool with CUR-101; minimum grade of 7.0 and 75% of activities. |

| academico-hands-on-002 | What are the criteria to complete this course? | Request for information that identifies the course. |

| academico-hands-on-003 | What are the completion criteria for the CUR-999 course? | Communication that the criteria are not available. |

| academico-hands-on-004 | What is the enrollment status for MAT-500? | Enrollment query without activating the new tool. |

In LangSmith, check the name, arguments, and result of the new tool. The first two turns should share the thread, while the others belong to separate conversations. In the case of CUR-999, the error is data returned by the query; it does not need to appear as an exception in the tracing.

-------------------------------------------------------------------TRADUÇÃO-------------------------------------------------------------


# Projeto de Agente Integrado como Assistente de Cursos

O projeto integra uma tool
Python local, uma tool que consulta uma API REST, tools de calendário e
critérios de conclusão descobertas em um servidor MCP, memória de curto prazo,
uma API FastAPI e tracing opcional no LangSmith.

## Requisitos

- Python 3.10 ou superior
- chave válida da API Gemini
- conta e chave do LangSmith apenas se o tracing for habilitado

## Como executar

1. Crie e ative um ambiente virtual.
2. Instale as dependências.

   ```bash
   pip install -r requirements.txt
   ```

3. Copie `.env.example` para `.env` e informe a chave do Gemini.
4. Inicie a API de catálogo em um terminal.

   ```bash
   fastapi dev catalogo_api.py --port 8001
   ```

5. Inicie a API do assistente em outro terminal.

   ```bash
   fastapi dev main.py --port 8000
   ```

6. Acesse `http://127.0.0.1:8000/docs`.

O servidor MCP é iniciado por `MCPAdapter` pelo transporte stdio. Ele não
precisa ser iniciado em um terceiro terminal.

## Como habilitar tracing

Altere as variáveis abaixo no arquivo `.env`.

```env
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=sua_chave_langsmith_aqui
LANGSMITH_PROJECT=pap-cap7-agente-integrado
```

Reinicie a API do assistente. As próximas execuções serão enviadas para o
projeto informado no LangSmith.

## Como validar

Execute os testes determinísticos.

```bash
python -m pytest -v
```

Com as duas APIs em execução, valide o agente e a continuidade da conversa.

```bash
python validar_integracao.py
```

## Hands-on de critérios de conclusão

A tool `consultar_criterios_conclusao` recebe `codigo_curso` e devolve os
critérios fictícios definidos em `dados.py`:

| Curso | Nota mínima (0 a 10) | Atividades concluídas (0 a 100%) |
| --- | --- | --- |
| CUR-101 | 7,0 | 75% |
| CUR-202 | 6,0 | 100% |

Esses dados descrevem regras do curso. A aplicação não possui notas ou
percentuais individuais para afirmar que um estudante já cumpriu os requisitos.

A consulta normaliza o código, valida o prefixo `CUR-` e devolve uma cópia do
registro encontrado. Um identificador inválido retorna
`{"erro": "código de curso inválido"}`. Um código sem registro retorna
`{"erro": "critérios de conclusão não encontrados"}`.

Execute os três testes da nova função:

```bash
python -m pytest tests/test_criterios.py -v
```

Eles verificam normalização e sucesso, código inválido e critérios não
cadastrados, sem chamadas ao modelo ou serviços em funcionamento. Para verificar
também os testes anteriores, execute `python -m pytest -v`.

Depois de alterar `mcp_server.py`, reinicie a API principal para que a descoberta
MCP inclua a nova tool. Mantenha o catálogo em funcionamento e o tracing
habilitado durante a validação integrada.

Envie as mensagens a seguir para `POST /chat` usando os campos
`conversation_id` e `message`. Mantenha a API principal em execução entre os
turnos.

| conversation_id | message | Verificação esperada |
| --- | --- | --- |
| academico-hands-on-001 | Minha matrícula é MAT-500. Em qual curso estou? | Identificação de CUR-101 e Python para Automação. |
| academico-hands-on-001 | Quais são os critérios para concluir esse curso? | Chamada à nova tool com CUR-101; nota mínima 7,0 e 75% das atividades. |
| academico-hands-on-002 | Quais são os critérios para concluir esse curso? | Solicitação de uma informação que identifique o curso. |
| academico-hands-on-003 | Quais são os critérios de conclusão do curso CUR-999? | Comunicação de que os critérios não estão disponíveis. |
| academico-hands-on-004 | Qual é o status da matrícula MAT-500? | Consulta de matrícula sem acionar a nova tool. |

No LangSmith, confira o nome, os argumentos e o resultado da nova tool. Os dois
primeiros turnos devem compartilhar a thread, enquanto os demais pertencem a
conversas separadas. No caso de CUR-999, o erro é um dado retornado pela consulta;
ele não precisa aparecer como uma exceção no tracing.


---------------------------------------------------------------TRADUCCIÓN----------------------------------------------------------------

# Proyecto de Agente Integrado como Asistente de Curso

El proyecto integra una herramienta local de Python, una herramienta que consulta una API REST, herramientas de calendario y criterios de finalización detectados en un servidor MCP, memoria a corto plazo, una API FastAPI y seguimiento opcional en LangSmith.

## Requisitos

- Python 3.10 o superior
- Clave API Gemini válida
- Cuenta y clave de LangSmith (solo si se habilita el seguimiento)

## Cómo ejecutar

1. Crea y activa un entorno virtual.

2. Instala las dependencias.

``bash
pip install -r requirements.txt

``

3. Copia `.env.example` a `.env` e introduce la clave Gemini.

4. Inicia la API del catálogo en una terminal.

```bash
fastapi dev catalogo_api.py --port 8001

``

5. Inicie la API del asistente en otra terminal.

``bash
fastapi dev main.py --port 8000

``

6. Acceda a `http://127.0.0.1:8000/docs`.

El servidor MCP se inicia mediante `MCPAdapter` a través del transporte stdio. No es necesario iniciarlo en una tercera terminal.

## Cómo habilitar el rastreo

Modifique las siguientes variables en el archivo `.env`.

``env LANGSMITH_TRACING=true LANGSMITH_API_KEY=su_clave_langsmith_aquí LANGSMITH_PROJECT=pap-cap7-integrated-agent
```

Reinicie la API del asistente. Las siguientes ejecuciones se enviarán al proyecto especificado en LangSmith.

## Cómo validar

Ejecute las pruebas deterministas.

``bash
python -m pytest -v

```

Con ambas API en ejecución, valide el agente y la continuidad de la conversación.

``bash
python validate_integration.py

```

## Criterios de finalización prácticos

La herramienta `consultar_criterios_conclusao` recibe `codigo_curso` y devuelve los criterios ficticios definidos en `dados.py`:

| Curso | Calificación mínima (0 a 10) | Actividades completadas (0 a 100%) |

| --- | --- | --- |

| CUR-101 | 7.0 | 75% |

| CUR-202 | 6.0 | 100% |

Estos datos describen las reglas del curso. La aplicación no dispone de calificaciones ni porcentajes individuales para indicar si un estudiante ha cumplido los requisitos.

La consulta normaliza el código, valida el prefijo `CUR-` y devuelve una copia del registro encontrado. Un identificador no válido devuelve ``{"error": "código de curso no válido"}`. Un código sin registro devuelve ``{"error": "criterios de finalización no encontrados"}`.

Ejecute las tres pruebas de la nueva función:

```bash
python -m pytest tests/test_criterios.py -v

```

Estas pruebas verifican la normalización y el éxito, el código no válido y los criterios no registrados, sin llamadas a modelos ni servicios en ejecución. Para verificar también las pruebas anteriores, ejecute `python -m pytest -v`.

Después de modificar `mcp_server.py`, reinicie la API principal para que el descubrimiento de MCP incluya la nueva herramienta. Mantenga el catálogo en ejecución y el rastreo habilitado durante la validación integrada.

Envíe los siguientes mensajes a `POST /chat` usando los campos `conversation_id` y `message`. Mantener la API principal en funcionamiento entre turnos.

| conversation_id | mensaje | Verificación esperada |

| --- | --- | --- |

| academico-hands-on-001 | Mi número de registro es MAT-500. ¿En qué curso estoy? | Identificación de CUR-101 y Python para automatización. |

| academico-hands-on-001 | ¿Cuáles son los criterios para completar este curso? | Llamada a la nueva herramienta con CUR-101; calificación mínima de 7.0 y 75% de las actividades. |

| academico-hands-on-002 | ¿Cuáles son los criterios para completar este curso? | Solicitud de información que identifique el curso. |

| academico-hands-on-003 | ¿Cuáles son los criterios de finalización para el curso CUR-999? | Comunicación de que los criterios no están disponibles. |

| academico-hands-on-004 | ¿Cuál es el estado de inscripción para MAT-500? | Consulta de inscripción sin activar la nueva herramienta.

En LangSmith, verifique el nombre, los argumentos y el resultado de la nueva herramienta. Los dos primeros turnos deben compartir el mismo hilo, mientras que los demás pertenecen a conversaciones separadas. En el caso de CUR-999, el error reside en los datos devueltos por la consulta; no es necesario que aparezca como una excepción en el rastreo.