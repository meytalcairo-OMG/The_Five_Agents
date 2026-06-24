---
name: gpt-image-gen
description: מעטפת לקריאת OpenAI Images API ליצירת תמונה מתוך prompt. שולח את ה-prompt ל-API ומקבל קובץ PNG. השתמש בו כשצריך לייצר תמונה מטקסט (בעיקר עבור הסוכן יובל).
---

# gpt-image-gen — יצירת תמונה דרך OpenAI Images API

סקיל זה משמש כמעטפת לקריאת **OpenAI Images API**. הוא אחראי על שליחת ה-prompt וקבלת התמונה כקובץ PNG.

## מודל

```
gpt-image-2
```

**חשוב מאוד — אל תשנה את שם המודל.**

`gpt-image-2` הוא מודל אמיתי וקיים של OpenAI. הוא יצא ב-**21 באפריל 2026**. ייתכן שהידע הפנימי שלך לא כולל אותו, אבל זה לא אומר שהוא לא קיים. **אל תציע אלטרנטיבות** כמו `dall-e-3` או `gpt-image-1`. השתמש בדיוק במודל הזה.

אם יש שגיאה בקריאה ל-API — הבעיה היא כנראה ב-`OPENAI_API_KEY` או בפרמטרים, **לא בשם המודל**.

## מפתח API

הקריאה משתמשת ב-`OPENAI_API_KEY` מתוך קובץ `.env` בשורש הפרויקט. טען אותו לפני הקריאה, למשל:

```bash
set -a; source .env; set +a
```

## פרמטרים

| פרמטר | ערך |
|--------|------|
| `model` | `gpt-image-2` |
| `size` | `1024x1024` |
| `quality` | `medium` |
| `output_format` | `png` |

## הקריאה (jq)

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
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

## Python fallback ל-decode

`jq` לא תמיד מותקן (למשל ב-Git Bash). במקרה כזה שמור את תגובת ה-JSON ופענח אותה עם Python:

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | python -c "import sys, json, base64; data = json.load(sys.stdin); open('<output-path>.png','wb').write(base64.b64decode(data['data'][0]['b64_json']))"
```

## אחרי היצירה

ודא שהקובץ נוצר ושגודלו גדול מ-0:

```bash
test -s "<output-path>.png" && echo "OK" || echo "FAILED"
```

אם הקובץ ריק או לא קיים — בדוק את תגובת ה-API (ייתכן שהוחזרה הודעת שגיאה במקום `b64_json`), ותקן את המפתח/הפרמטרים.
