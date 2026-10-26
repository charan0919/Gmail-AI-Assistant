\# 📧 Gmail AI Assistant



An intelligent email assistant powered by Google Gemini AI and LangChain that helps you search, analyze, and get insights from your Gmail inbox using natural language queries.



\## ✨ Features



\- 🔍 Semantic search with natural language queries

\- 🤖 AI-powered contextual responses

\- 📊 Email analytics and search history

\- 🎨 Modern Streamlit interface

\- 🔒 Local processing for privacy

\- 📝 Multi-email context understanding



\## 🏗️ Architecture



```

Gmail → OAuth2 → Gmail API → Fetch \& Process Emails

&#x20;                                     ↓

&#x20;                         HuggingFace Embeddings (all-MiniLM-L6-v2)

&#x20;                                     ↓

&#x20;                              FAISS Vector Store

&#x20;                                     ↓

&#x20;                   User Query → Similarity Search → Top-K Emails

&#x20;                                     ↓

&#x20;                           Google Gemini AI → Response

```



\## 🔬 Technical Architecture



\### Core Components



\*\*1. Data Preprocessing (`data\_preprocess.py`)\*\*

\- \*\*Gmail API\*\*: OAuth 2.0 authentication, fetches messages with `gmail.readonly` scope

\- \*\*Content Processing\*\*: 

&#x20; - Recursive MIME parsing (handles multipart/alternative, text/plain, text/html)

&#x20; - BeautifulSoup4 for HTML cleaning

&#x20; - Base64 decoding for email bodies

&#x20; - 4000 char truncation for context management

\- \*\*Document Format\*\*:

```python

Document(

&#x20;   page\_content="Email body...",

&#x20;   metadata={"subject", "from", "date", "id", "labels", "email\_number"}

)

```



\*\*2. Embedding Generation\*\*

\- \*\*Model\*\*: all-MiniLM-L6-v2 (Sentence-BERT)

&#x20; - 384-dimensional vectors

&#x20; - \~14,000 sentences/sec on CPU

&#x20; - 80MB model size

&#x20; - Optimized for semantic similarity

\- \*\*Process\*\*: Tokenization → Mean pooling → L2 normalization



\*\*3. Vector Database: FAISS\*\*

\- \*\*Index\*\*: IndexFlatL2 (exact L2 distance search)

\- \*\*Algorithm\*\*: Brute-force nearest neighbors, O(n) complexity

\- \*\*Storage\*\*: Binary index + pickled metadata

\- \*\*Metric\*\*: Cosine similarity via normalized L2 distance



\*\*4. RAG Pipeline\*\*

```

Query → Embedding → FAISS Search → Top-K Emails → Context → Gemini → Response

```



\*\*5. LLM: Google Gemini\*\*

\- \*\*Model\*\*: gemini-2.0-flash-exp

\- \*\*Config\*\*: Temperature 0.5, Max tokens 2048

\- \*\*Context\*\*: 1M token window (uses \~8-10K per request)

\- \*\*Prompt Structure\*\*: System role + User query + Email context + Instructions



\*\*6. Streamlit Application (`app.py`)\*\*

\- \*\*Caching\*\*: `@st.cache\_resource` for models, session state for user data

\- \*\*Components\*\*: Sidebar (settings) + Main (stats, query, results, AI response, history)



\### Data Flow



```

User Query → Streamlit → Embedding (all-MiniLM-L6-v2) → FAISS Search

&#x20;   → Retrieved Emails → Prompt Construction → Gemini API → Response Rendering

```



\### Performance



\- \*\*Indexing\*\*: \~0.5-1 sec/email, 100 emails in 2-3 min

\- \*\*Query\*\*: Embedding (50ms) + FAISS (10-50ms) + Gemini (1-3s) = \*\*2-4 sec total\*\*

\- \*\*Scalability\*\*: Optimal for <1000 emails, \~10MB per 1000 emails



\### Technology Stack



```yaml

Backend:

&#x20; - langchain 0.1.0, langchain-google-genai 0.0.6

&#x20; - faiss-cpu 1.8.0, sentence-transformers 2.2.2

&#x20; - google-api-python-client 2.108.0, google-auth 2.25.2



Frontend:

&#x20; - streamlit 1.29.0



Utilities:

&#x20; - beautifulsoup4 4.12.2, python-dotenv 1.0.1

```



\### Security \& Privacy



\- \*\*Auth\*\*: OAuth 2.0 with token refresh, no password storage

\- \*\*Data\*\*: All processing local, only query+context sent to Gemini

\- \*\*Secrets\*\*: Environment variables, credentials excluded from git



\### Limitations



\- Context limited to \~10 emails per query

\- Optimized for <1000 emails

\- Manual re-indexing required for new emails

\- Text-only (no image/attachment processing)



\## 🚀 Installation



```bash

\# Clone and setup

cd gmail-ai-assistant

python -m venv venv

source venv/bin/activate  # Windows: venv\\Scripts\\activate

pip install -r requirements.txt

```



\## 🔑 API Setup



\### Gmail API



1\. \*\*Google Cloud Console\*\* → Create project → Enable Gmail API

2\. \*\*OAuth Consent\*\*: External → Add app details → Scopes: `gmail.readonly` → Test users: your email

3\. \*\*Credentials\*\*: Create OAuth Client ID → Desktop app → Download JSON → Rename to `credentials.json`



\### Gemini API



1\. \*\*Google AI Studio\*\* → Create API key

2\. Create `.env` file:

```env

GOOGLE\_API\_KEY=your\_key\_here

```



\## 🎯 Usage



```bash

\# Index emails (first time)

python data\_preprocess.py



\# Launch app

streamlit run app.py

```



\*\*First OAuth Run\*\*: Browser opens → Sign in → "Advanced" → "Go to app (unsafe)" → Continue



\*\*Example Queries\*\*:

\- "Emails about internships?"

\- "Show emails from Amazon"

\- "What did John say about the project?"



\## ⚙️ Configuration



\*\*Index more emails\*\*: Edit `data\_preprocess.py`

```python

MAX\_EMAILS = 100  # Change to 200, 500, etc.

```



\*\*Retrieval settings\*\*: Adjust slider in app sidebar (3-15 emails)



\*\*Different embedding model\*\*: Change `MODEL\_NAME` in both files

\- `all-mpnet-base-v2` (better quality, slower)

\- `paraphrase-MiniLM-L6-v2` (faster)



\## 🔧 Troubleshooting



| Issue | Solution |

|-------|----------|

| Vector store not found | Run `python data\_preprocess.py` |

| API key not found | Check `.env` file exists with correct format |

| OAuth fails | Delete `token.json`, verify test user added |

| No emails found | Check Gmail has emails, increase `MAX\_EMAILS` |

| Empty responses | Verify Gemini API key valid, check quota |



\## 📊 Performance Tips



\- Start with 100 emails, increase gradually

\- Use specific queries with context (dates, names)

\- Reduce `top\_k` in sidebar for faster responses

\- Re-index periodically for new emails



\## 🔒 Security Best Practices



\*\*Add to .gitignore\*\*:

```

.env

credentials.json

token.json

faiss\_index/

```



\- Never share API keys or OAuth credentials

\- Keep dependencies updated: `pip install --upgrade -r requirements.txt`

\- Rotate API keys quarterly



\## 📝 Project Structure



```

gmail-ai-assistant/

├── app.py                  # Streamlit app

├── data\_preprocess.py      # Email indexing

├── requirements.txt        # Dependencies

├── .env                    # API keys (create)

├── credentials.json        # OAuth creds (download)

├── token.json             # Auto-generated

└── faiss\_index/           # Auto-generated

```



\## 🎓 Resources



\- \[LangChain Docs](https://python.langchain.com/)

\- \[Gmail API Docs](https://developers.google.com/gmail/api)

\- \[Gemini API Docs](https://ai.google.dev/docs)

\- \[FAISS GitHub](https://github.com/facebookresearch/faiss)



\---



\*\*Made with ❤️ using Streamlit, LangChain \& Google Gemini\*\*



🔒 \*Emails processed locally. Only queries sent to Gemini API.\*

