# AdSnap Studio - Architecture Documentation

## Overview
AdSnap Studio is a Streamlit-based application for generating professional product advertisements using Bria AI's image generation and manipulation APIs.

---

## 1. Entry Point

**File:** `app.py`  
**Command:** `streamlit run app.py`

### Initialization Flow:
1. Load environment variables from `.env` file
2. Initialize Streamlit page configuration
3. Load `BRIA_API_KEY` environment variable
4. Initialize session state (API key, images, prompts)
5. Render main UI with tabs

---

## 2. Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `BRIA_API_KEY` | Authentication token for Bria AI API | ✓ Yes |

**Location:** `.env` file (root directory)

---

## 3. UI Tabs & Service Architecture

### Tab 1: 🎨 Generate Image
**Services Used:**
- `services/hd_image_generation.py` → `generate_hd_image()`
- `services/prompt_enhancement.py` → `enhance_prompt()`

**Features:**
- Text prompt input
- AI prompt enhancement
- Image style selection (Realistic, Artistic, Cartoon, etc.)
- Aspect ratio selection
- Number of images (1-4)
- Quality enhancement option

**Bria Endpoints:**
- `POST /v1/text-to-image/hd/{model_version}` (default: 2.2)
- `POST /v1/prompt_enhancer`

---

### Tab 2: 🖼️ Lifestyle Shot
**Services Used:**
- `services/lifestyle_shot.py` → `lifestyle_shot_by_text()` or `lifestyle_shot_by_image()`
- `services/packshot.py` → `create_packshot()`
- `services/shadow.py` → `add_shadow()`

**Features:**
- Upload product image
- Create packshot (background removal & standardization)
- Add shadow effects (Natural/Drop, with adjustable intensity)
- Generate lifestyle shots with text or reference image
- Multiple placement options (Original, Automatic, Manual)

**Bria Endpoints:**
- `POST /v1/product/packshot`
- `POST /v1/product/shadow`
- `POST /v1/product/lifestyle_shot_by_text`
- `POST /v1/product/lifestyle_shot_by_image`

---

### Tab 3: 🎨 Generative Fill
**Services Used:**
- `services/generative_fill.py` → `generative_fill()`
- `streamlit_drawable_canvas` → drawing canvas component

**Features:**
- Upload image
- Draw mask on image using canvas
- Describe content to generate in masked area
- Optional negative prompt
- Seed for reproducibility
- Content moderation toggle

**Bria Endpoint:**
- `POST /v1/gen_fill`

---

### Tab 4: 🎨 Erase Elements
**Services Used:**
- `services/erase_foreground.py` → `erase_foreground()`

**Features:**
- Upload product image
- Remove foreground and generate background
- Content moderation option

**Bria Endpoint:**
- `POST /v1/erase_foreground`

---

## 4. API Call Flow

### Typical Flow for Image Generation:

```
User Input
    ↓
Session State Management
    ↓
Service Function Call (e.g., generate_hd_image)
    ↓
Prepare Request Data
    ├─ API Key (from BRIA_API_KEY)
    ├─ Image/Mask data (base64 encoded)
    ├─ Parameters (prompt, style, etc.)
    └─ Options (sync/async, moderation, etc.)
    ↓
HTTP POST to Bria Endpoint
    ├─ URL: https://engine.prod.bria-api.com/v1/...
    ├─ Header: api_token
    └─ JSON: request data
    ↓
Response Processing
    ├─ Sync Mode: Wait for result_url
    └─ Async Mode: Get pending URLs
    ↓
Display Result / Download Option
```

---

## 5. Data Encoding

### Image Input Handling:
- **Direct Upload:** PIL Image → bytes → base64 string
- **URL Input:** Direct string (some endpoints support this)

### Base64 Encoding Used In:
- `services/packshot.py` (image file)
- `services/shadow.py` (image file)
- `services/lifestyle_shot.py` (image & reference file)
- `services/generative_fill.py` (image & mask file)
- `services/erase_foreground.py` (image file)

---

## 6. Session State Management

**Session Variables:**
```python
st.session_state.api_key          # User's API key
st.session_state.generated_images # List of generated image URLs
st.session_state.current_image    # Currently viewing image
st.session_state.pending_urls     # URLs pending generation
st.session_state.edited_image     # Current edited image URL
st.session_state.original_prompt  # Original user prompt
st.session_state.enhanced_prompt  # AI-enhanced prompt
```

---

## 7. File Structure

```
adsnap-studio/
├── app.py                          # Main Streamlit application
├── requirements.txt                # Python dependencies
├── .env                           # Environment variables (BRIA_API_KEY)
├── README.md                      # User documentation
├── ARCHITECTURE.md                # This file
├── components/
│   ├── __init__.py
│   ├── image_preview.py          # Image display component
│   ├── sidebar.py                # Sidebar configuration
│   └── uploader.py               # File upload component
├── services/
│   ├── __init__.py               # Service exports
│   ├── hd_image_generation.py    # Text-to-image generation
│   ├── prompt_enhancement.py     # AI prompt enhancement
│   ├── packshot.py               # Product packshot creation
│   ├── shadow.py                 # Shadow addition
│   ├── lifestyle_shot.py         # Lifestyle shot generation
│   ├── generative_fill.py        # Mask-based content generation
│   └── erase_foreground.py       # Foreground removal
└── utils/                         # Utility functions (if needed)
```

---

## 8. Bria API Endpoints Summary

| Endpoint | Purpose | Service File |
|----------|---------|--------------|
| `/v1/text-to-image/hd/{version}` | Generate HD images from text | `hd_image_generation.py` |
| `/v1/prompt_enhancer` | Enhance text prompts | `prompt_enhancement.py` |
| `/v1/product/packshot` | Create professional packshots | `packshot.py` |
| `/v1/product/shadow` | Add shadows to products | `shadow.py` |
| `/v1/product/lifestyle_shot_by_text` | Generate lifestyle context (text) | `lifestyle_shot.py` |
| `/v1/product/lifestyle_shot_by_image` | Generate lifestyle context (image) | `lifestyle_shot.py` |
| `/v1/gen_fill` | Generate content in masked area | `generative_fill.py` |
| `/v1/erase_foreground` | Remove and regenerate foreground | `erase_foreground.py` |

---

## 9. Error Handling

- All service functions raise exceptions on API errors
- Streamlit UI catches exceptions and displays user-friendly error messages
- Optional `streamlit-drawable-canvas` has fallback error handling in generative fill tab

---

## 10. Dependencies

**Core:**
- `streamlit==1.32.0` - Web framework
- `requests==2.31.0` - HTTP requests
- `Pillow==10.2.0` - Image processing
- `python-dotenv==1.0.1` - Environment management
- `python-magic==0.4.27` - File type detection

**Optional:**
- `streamlit-drawable-canvas` - Drawing canvas component (for Generative Fill)

---

## 11. Key Design Patterns

### Service Layer Pattern:
Each service module encapsulates a specific Bria API feature with:
- Typed function signatures
- Parameter validation
- Request preparation (base64 encoding, etc.)
- Error handling and logging
- Response parsing

### Session State Management:
UI state persisted across Streamlit reruns using `st.session_state`

### Async Image Handling:
- Sync mode: Wait for immediate results
- Async mode: Get URLs, poll for completion

---

## 12. Security Considerations

⚠️ **Important:**
- API key stored in `.env` (never commit to version control)
- `.env` file should be in `.gitignore`
- API key transmitted via HTTP headers to Bria API
- Base64 encoding is for API transmission format (not encryption)

