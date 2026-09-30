# ComicCraft — AI Comic Story Creator

ComicCraft is a FastAPI + Jinja2 application based on the supplied project specification. It generates a five-panel comic through:

1. Gemini structured outline generation
2. Gemini story/narration/dialogue generation
3. Stable Diffusion image generation
4. Panel layout assembly
5. FPDF PDF export

## Important implementation note

The supplied document names `gemini-1.5-flash` and `gemini-1.5-pro`. This implementation uses the current Google GenAI Python SDK and configurable model names instead of the deprecated `google-generativeai` SDK. The default models are configured in `.env`.

For a quick first run, `IMAGE_BACKEND=placeholder` avoids downloading a multi-GB local Stable Diffusion model. After the text pipeline works, switch to `IMAGE_BACKEND=diffusers` for local Stable Diffusion generation.

## Windows / VS Code

### 1. Open the folder

Open the `ComicCraft` folder in VS Code.

### 2. Create a virtual environment

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

If PowerShell blocks activation, use Command Prompt:

```cmd
.venv\Scripts\activate
```

### 3. Install dependencies

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

For a CPU-only first run, the default PyTorch package may be large. If you already have a suitable PyTorch installation, keep it.

### 4. Configure secrets

Copy `.env.example` to `.env`:

```powershell
copy .env.example .env
```

Put your Gemini key in `.env`:

```text
GEMINI_API_KEY=your_real_key
```

Never commit `.env` or share your Gemini key.

### 5. Start the application

```powershell
uvicorn app.main:app --reload
```

Open:

- http://127.0.0.1:8000
- http://127.0.0.1:8000/docs
- http://127.0.0.1:8000/health

## First test

Keep:

```text
IMAGE_BACKEND=placeholder
```

Create a comic from the browser. This verifies FastAPI, Gemini, Jinja2, routing, layout, and PDF export without requiring a local image model.

## Enable Stable Diffusion

After the first test succeeds:

```text
IMAGE_BACKEND=diffusers
```

Then restart Uvicorn.

The first image generation downloads the configured Stable Diffusion model and can require substantial disk space and RAM/VRAM. A CUDA-capable NVIDIA GPU is strongly recommended for practical local generation.

## API test

In Swagger at `/docs`, POST to `/generate-comic/json` with:

```json
{
  "story_prompt": "A brave fox exploring an enchanted forest",
  "character_name": "Milo",
  "setting": "Enchanted forest",
  "tone": "Funny",
  "art_style": "Comic book"
}
```

Or with PowerShell:

```powershell
$body = @{
  story_prompt = "A brave fox exploring an enchanted forest"
  character_name = "Milo"
  setting = "Enchanted forest"
  tone = "Funny"
  art_style = "Comic book"
} | ConvertTo-Json

Invoke-RestMethod `
  -Uri http://127.0.0.1:8000/generate-comic/json `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

## Test image endpoint

```text
http://127.0.0.1:8000/test-image?prompt=A%20brave%20fox%20in%20an%20enchanted%20forest
```

## Troubleshooting

### `GEMINI_API_KEY is not configured`
Make sure `.env` exists in the project root and contains a valid key. Restart Uvicorn after editing `.env`.

### Gemini quota / 429
The application cannot bypass a Google API quota. Check the API project, billing/quota configuration, and model availability for your key.

### Diffusers is slow
This is expected on CPU. Use a CUDA-enabled PyTorch installation and compatible NVIDIA GPU if you want practical local Stable Diffusion generation.

### PDF export fails
Check that the generated images exist under `static/panels`. The application creates `static/panels` and `static/exports` automatically.

## Project structure

```text
ComicCraft/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── routes.py
│   ├── config.py
│   ├── schemas.py
│   ├── gemini_flash.py
│   ├── gemini_pro.py
│   ├── image_generator.py
│   ├── layout_builder.py
│   └── exporters.py
├── static/
│   ├── css/style.css
│   ├── panels/
│   └── exports/
├── templates/
│   ├── base.html
│   ├── index.html
│   ├── comic_preview.html
│   └── export_success.html
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```
