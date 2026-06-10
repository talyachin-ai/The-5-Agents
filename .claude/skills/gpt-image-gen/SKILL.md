---
name: gpt-image-gen
description: Use when you need to generate an image from a text prompt via OpenAI Images API. Sends the prompt and returns a saved PNG file. Used primarily by the יובל (Yuval) image-designer agent.
---

# gpt-image-gen — מעטפת ליצירת תמונות (OpenAI Images API)

הסקיל הזה אחראי על דבר אחד: לקחת `prompt` טקסטואלי, לשלוח אותו ל-OpenAI Images API,
ולשמור את התמונה שחזרה כקובץ PNG. הוא לא מנסח prompts ולא בוחר סגנון — זה התפקיד של יובל.

## המודל — `gpt-image-2`

> ⚠️ **אל תשנה את שם המודל. השתמש בדיוק ב-`gpt-image-2`.**
>
> - `gpt-image-2` הוא מודל **אמיתי וקיים** של OpenAI, שיצא ב-**21 באפריל 2026**.
> - ייתכן שהוא לא נמצא בידע הפנימי של המודל שמריץ את הסקיל — **זה לא אומר שהוא לא קיים**.
> - **אין** להציע או לעבור לאלטרנטיבות (`dall-e-3`, `gpt-image-1`, וכו').
> - אם מתקבלת שגיאה מה-API, הבעיה כמעט תמיד היא ב-**API key** או ב-**parameters** — **לא** בשם המודל.

## דרישות

- `OPENAI_API_KEY` מוגדר ב-`.env` בשורש הפרויקט.
- הרצה דרך **Bash** (הקריאה מבוססת `curl`).
- `python` זמין (משמש כ-fallback ל-decode כש-`jq` לא מותקן — נפוץ ב-Git Bash על Windows).

## פרטי הקריאה

- **Endpoint:** `POST https://api.openai.com/v1/images/generations`
- **Auth:** `Authorization: Bearer $OPENAI_API_KEY`
- **Parameters:**

| param | value |
|-------|-------|
| `model` | `gpt-image-2` |
| `prompt` | `<the prompt>` |
| `size` | `1024x1024` |
| `quality` | `medium` |
| `output_format` | `png` |

- **הפלט** חוזר בשדה `data[0].b64_json` (base64 — **לא** URL), ולכן צריך לפענח אותו לקובץ.

## נתיב primary (עם jq)

טען קודם את `OPENAI_API_KEY` מ-`.env`:

```bash
export OPENAI_API_KEY="$(grep -E '^OPENAI_API_KEY=' .env | cut -d '=' -f2-)"
```

```bash
curl -sS -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

## נתיב fallback (Python — כש-jq לא מותקן)

שומרים את תשובת ה-JSON לקובץ זמני, ואז מפענחים עם Python (שתמיד זמין ב-Git Bash):

```bash
curl -sS -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > /tmp/gpt-image-resp.json

python -c "import json,base64; d=json.load(open('/tmp/gpt-image-resp.json')); open('<output-path>.png','wb').write(base64.b64decode(d['data'][0]['b64_json']))"
```

> אם ה-JSON מכיל שדה `error` במקום `data`, הדפיסו אותו — זו האבחנה (key/parameters), **לא** שם המודל:
> `python -c "import json; print(json.load(open('/tmp/gpt-image-resp.json')).get('error'))"`

## אימות

אחרי השמירה, ודאו שהקובץ קיים וגדול מ-0:

```bash
test -s "<output-path>.png" && echo "OK: $(wc -c < '<output-path>.png') bytes" || echo "FAILED: empty or missing"
```
