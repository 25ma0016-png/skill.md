---
name: writing-evaluator-app
description: Builds a complete, beginner-friendly Python + Streamlit "Automated Writing Scoring and Evaluation System" that scores uploaded TXT/DOCX essays out of 100 (grammar & spelling 25, vocabulary 20, sentence structure 20, content & relevance 25, punctuation 10) using free LanguageTool and simple NLP, with remarks, strengths, suggestions, highlighted issues and downloadable TXT/Word reports. Use this skill whenever the user asks for an essay grader, automated writing evaluation/scoring, AES project, writing feedback tool, grammar-scoring web app, or a student/college project that rates writing, even if they don't name Streamlit or LanguageTool.
---

# Automated Writing Scoring and Evaluation System

This skill contains a complete, tested project. Recreate it exactly from the source files at the end of this document, then adapt it only where the user asks.

## What the app does

1. The user uploads a `.txt` or `.docx` file in a Streamlit web page.
2. `file_reader.py` extracts and cleans the text, rejecting empty, wrong-type, damaged or oversized files with friendly messages.
3. `grammar_checker.py` checks grammar, spelling and punctuation with LanguageTool. It tries a local LanguageTool (needs Java 17+), then the free public server, then built-in basic rules, so the app never crashes.
4. `text_analyzer.py` performs simple NLP with regular expressions: paragraph/sentence/word tokenization, moving-average type-token ratio (lexical diversity), sentence-length statistics, transition words, keywords and readability (textstat).
5. `scorer.py` converts the measurements into marks with readable `if` rules and linear `scale()` mappings, then builds the remark, overall comment, strengths and suggestions.
6. `app.py` scores automatically after upload (no button), shows a red-pen style total, category bars, tabs (Feedback, Detailed analysis, Issues found, Your text with highlights) and download buttons.
7. `report_generator.py` builds the TXT and Word reports.

No paid APIs or API keys are used anywhere.

## Workflow when using this skill

1. Create the folder structure below and write every file exactly as given in "Complete source files". Create `sample_files/empty_file.txt` as a zero-byte file.
2. If the user asks for changes (new categories, different marks, other languages), edit `scorer.py` / `MAX_SCORES` and keep the category maxima summing to 100.
3. Test before delivering when an environment is available:
   - `pip install -r requirements.txt`
   - Run `evaluate_writing()` on both sample files with `GrammarChecker("en-GB")`. Expected: the good essay scores around 90+ (Excellent), the mistakes essay around 40 (Needs Improvement).
   - Call `read_uploaded_file()` with fake objects (`name`, `getvalue()`) for an empty file, a `.pdf`, a `.doc` and a fake `.docx` and confirm each raises `FileReadError`.
   - Optionally run the app with `streamlit.testing.v1.AppTest`.
4. Deliver the files plus clear, simple-language setup instructions, and always state that the score is an automated estimate, not a perfect human evaluation.

## Folder structure

```
writing_evaluator/
├── app.py                     # Streamlit user interface
├── requirements.txt           # Python libraries
├── README.md                  # Beginner guide
├── .streamlit/config.toml     # Theme + 5 MB upload limit
├── utils/
│   ├── __init__.py
│   ├── file_reader.py         # TXT/DOCX reading and upload validation
│   ├── text_analyzer.py       # NLP measurements
│   ├── grammar_checker.py     # LanguageTool + fallbacks
│   ├── scorer.py              # Marking scheme
│   └── report_generator.py    # TXT and Word reports
└── sample_files/
    ├── sample_essay.txt
    ├── sample_with_mistakes.txt
    └── empty_file.txt          # zero bytes
```

## Marking scheme

| Category | Marks | Rule summary |
|---|---|---|
| Grammar & Spelling | 25 | Errors per 100 words: 0 → 25 marks, 8+ → 0 (linear) |
| Vocabulary | 20 | Lexical diversity 0.55→0.80 (10), long-word ratio 8%→25% (6), overused words (4 minus count) |
| Sentence Structure | 20 | Average length 12–22 words (8), length spread + varied openings (6), few run-ons >40 / fragments <4 (6) |
| Content & Relevance | 25 | Word count 60→300 (8), paragraphs (5), linking words (5), topic keyword coverage or estimated focus (7) |
| Punctuation | 10 | Punctuation/casing errors per 100 words (5), paragraphs ending properly (2), commas in longer sentences (2), few "!" (1) |

Remarks: Excellent ≥ 90, Very Good ≥ 75, Good ≥ 60, Average ≥ 45, otherwise Needs Improvement. Text must be 30–5,000 words.

## Important implementation notes

- `language_tool_python` 3.x uses snake_case match fields (`rule_id`, `rule_issue_type`, `error_length`); 2.x uses camelCase. The checker reads both with `getattr` fallbacks. Keep that.
- Style-only and locale rules (`EN_QUOTES`, `DASH_RULE`, issue types `style`, `locale-violation`) are ignored so they don't lower scores unfairly.
- Offer British/American English in the sidebar; Indian and UK students usually need `en-GB`.
- `@st.cache_resource` keeps one LanguageTool instance; `@st.cache_data` stores results per text/topic/language. The first run downloads LanguageTool (~250 MB) and shows a spinner explaining that.
- Short headings (≤12 words, no end punctuation) are excluded from sentence statistics but included in topic relevance.
- Use `width="stretch"` (not the deprecated `use_container_width`) and require `streamlit>=1.50`.

## Setup instructions to give the user

1. Install Python 3.9+ and Java 17+ (free from adoptium.net). Check with `python --version` and `java -version`.
2. Open a terminal in the `writing_evaluator` folder.
3. `python -m venv venv`
4. Activate: `venv\Scripts\activate` (Windows) or `source venv/bin/activate` (Mac/Linux).
5. `pip install -r requirements.txt`
6. `streamlit run app.py` and open http://localhost:8501.
7. Upload `sample_files/sample_essay.txt` to try it.

## Troubleshooting

- "LanguageTool could not be started" warning: install Java 17+ and restart; the app meanwhile uses the online server or basic checks.
- `streamlit` not recognised: activate the virtual environment, or run `python -m streamlit run app.py`.
- Slow first run: LanguageTool is downloading once.

## Required disclaimer

Always include this message in the app, the reports and the explanation: the score is an automated estimate, not a perfect human evaluation. It cannot fully judge creativity, argument quality, tone or factual accuracy, so it should be used alongside feedback from a teacher or reviewer.

---

# Complete source files

Write each file exactly as shown.

## `requirements.txt`

~~~~text
streamlit>=1.50
language-tool-python>=2.8
python-docx>=1.1
textstat>=0.7.3
pandas>=2.0
~~~~

## `.streamlit/config.toml`

~~~~toml
[theme]
base = "light"
primaryColor = "#1F3A5F"
backgroundColor = "#FBFBF8"
secondaryBackgroundColor = "#EEF2F6"
textColor = "#1C2430"

[server]
maxUploadSize = 5
~~~~

## `utils/__init__.py`

~~~~python
"""Helper modules for the Automated Writing Scoring and Evaluation System."""
~~~~

## `utils/file_reader.py`

~~~~python
"""
file_reader.py
--------------
Reads the text out of an uploaded TXT or DOCX file.

Every problem (wrong file type, empty file, broken DOCX, ...) is turned into a
FileReadError with a friendly message, so the app can show it to the user
instead of crashing.
"""

import io
import re

from docx import Document

ALLOWED_EXTENSIONS = ("txt", "docx")
MAX_FILE_SIZE_MB = 5


class FileReadError(Exception):
    """Raised when an uploaded file cannot be used for evaluation."""


def read_uploaded_file(uploaded_file):
    """Return the cleaned text inside a Streamlit UploadedFile object."""
    if uploaded_file is None:
        raise FileReadError("No file was uploaded. Please choose a TXT or DOCX file.")

    # 1. Check the file extension
    file_name = uploaded_file.name
    extension = file_name.rsplit(".", 1)[-1].lower() if "." in file_name else ""
    if extension == "doc":
        raise FileReadError(
            "Old Word files (.doc) are not supported. "
            "Open the file in Word and use 'Save As' → .docx, then upload it again."
        )
    if extension not in ALLOWED_EXTENSIONS:
        raise FileReadError(
            f"'{file_name}' is not a supported file. Please upload a .txt or .docx file."
        )

    # 2. Check the file size
    data = uploaded_file.getvalue()
    if len(data) == 0:
        raise FileReadError("The uploaded file is empty. Please upload a file that contains writing.")
    if len(data) > MAX_FILE_SIZE_MB * 1024 * 1024:
        raise FileReadError(f"The file is larger than {MAX_FILE_SIZE_MB} MB. Please upload a smaller file.")

    # 3. Extract the text
    if extension == "txt":
        text = _read_txt(data)
    else:
        text = _read_docx(data)

    text = clean_text(text)
    if not text.strip():
        raise FileReadError("The file does not contain any readable text.")
    return text


def _read_txt(data):
    """Decode a plain-text file, trying the most common encodings."""
    # Files saved as UTF-16 (e.g. some Windows Notepad files) start with a "BOM" marker
    if data.startswith(b"\xff\xfe") or data.startswith(b"\xfe\xff"):
        return data.decode("utf-16")
    for encoding in ("utf-8-sig", "cp1252"):
        try:
            return data.decode(encoding)
        except UnicodeDecodeError:
            continue
    # latin-1 can decode any byte, so it is our last resort
    return data.decode("latin-1")


def _read_docx(data):
    """Read the paragraphs (and any table text) from a Word .docx file."""
    try:
        document = Document(io.BytesIO(data))
    except Exception:
        raise FileReadError(
            "This DOCX file could not be opened. It may be damaged, password-protected, "
            "or not a real Word document."
        )

    parts = [p.text for p in document.paragraphs if p.text.strip()]

    # Some people write inside tables, so read those too
    for table in document.tables:
        for row in table.rows:
            for cell in row.cells:
                if cell.text.strip():
                    parts.append(cell.text)

    # A blank line between paragraphs lets the analyser find paragraph breaks
    return "\n\n".join(parts)


def clean_text(text):
    """Tidy up spacing and line breaks without changing the writing itself."""
    text = text.replace("\r\n", "\n").replace("\r", "\n").replace("\t", " ")
    text = text.replace("\u00a0", " ")  # non-breaking spaces
    lines = [line.rstrip() for line in text.split("\n")]
    text = "\n".join(lines)
    text = re.sub(r"\n{3,}", "\n\n", text)  # at most one blank line in a row
    return text.strip()
~~~~

## `utils/text_analyzer.py`

~~~~python
"""
text_analyzer.py
----------------
Basic NLP (Natural Language Processing) for the uploaded writing.

It splits the text into paragraphs, sentences and words (this is called
"tokenization") and then measures things such as:
  * how varied the vocabulary is (lexical diversity)
  * how long the sentences are
  * which linking/transition words are used
  * the main keywords of the text
  * how easy the text is to read (readability)

Plain Python and regular expressions are used, so no large model downloads
are needed.
"""

import re
import statistics
from collections import Counter

import textstat

# Very common words that carry little meaning ("stop words")
STOPWORDS = set("""
a about above after again against all also am an and any are as at be because been before
being below between both but by can cannot could did do does doing down during each few for
from further get got had has have having he her here hers herself him himself his how i if
in into is it its itself just let like make made many may me might more most much must my
myself no nor not now of off on once one only or other our ours ourselves out over own same
she should so some such than that the their theirs them themselves then there these they
this those through to too two under until up upon us very was we well were what when where
which while who whom why will with would yet you your yours yourself yourselves
""".split())

# Linking words that help ideas flow from one to the next
TRANSITION_WORDS = [
    "however", "therefore", "moreover", "furthermore", "in addition", "additionally",
    "consequently", "as a result", "for example", "for instance", "in conclusion",
    "to conclude", "in summary", "to sum up", "on the other hand", "nevertheless",
    "nonetheless", "meanwhile", "similarly", "likewise", "in contrast", "firstly",
    "secondly", "thirdly", "finally", "first of all", "in fact", "thus", "hence",
    "although", "even though", "besides", "overall", "to begin with", "in other words",
    "instead", "otherwise", "specifically", "in particular", "ultimately", "afterwards",
    "subsequently", "on the contrary", "as well as", "whereas", "lastly", "above all",
]

# Abbreviations whose full stop should not end a sentence
ABBREVIATIONS = ["Mr.", "Mrs.", "Ms.", "Dr.", "Prof.", "Sr.", "Jr.", "St.", "vs.",
                 "etc.", "e.g.", "i.e.", "a.m.", "p.m.", "U.S.", "U.K.", "No."]

SENTENCE_END = (".", "!", "?")
CLOSING_MARKS = "\"'”’)]"


def tokenize_words(text):
    """Split text into words, keeping contractions like "don't" together."""
    return re.findall(r"[A-Za-z]+(?:['’][A-Za-z]+)*", text)


def split_paragraphs(text):
    """Split text into paragraphs (blocks separated by blank lines)."""
    blocks = [b.strip() for b in re.split(r"\n\s*\n", text) if b.strip()]

    # Some TXT files use one line per paragraph with no blank lines in between.
    if len(blocks) == 1:
        lines = [line.strip() for line in text.split("\n") if line.strip()]
        ends_like_paragraph = sum(1 for line in lines if line.rstrip(CLOSING_MARKS).endswith(SENTENCE_END))
        if len(lines) > 1 and ends_like_paragraph >= 0.6 * len(lines):
            blocks = lines

    # Join lines that were wrapped inside the same paragraph
    return [" ".join(line.strip() for line in block.split("\n")) for block in blocks]


def is_heading(paragraph):
    """A short line with no full stop at the end is treated as a title/heading."""
    words = tokenize_words(paragraph)
    ends_with_punctuation = paragraph.rstrip(CLOSING_MARKS).endswith(SENTENCE_END)
    return len(words) <= 12 and not ends_with_punctuation


def split_sentences(paragraph):
    """Split a paragraph into sentences at . ! or ? followed by a space."""
    protected = paragraph
    for abbreviation in ABBREVIATIONS:
        pattern = r"(?<![A-Za-z])" + re.escape(abbreviation)
        protected = re.sub(pattern, lambda m: m.group(0).replace(".", "§"),
                           protected, flags=re.IGNORECASE)

    pieces = re.split(r"(?<=[.!?][\"'”’)\]])\s+|(?<=[.!?])\s+", protected)
    sentences = [p.replace("§", ".").strip() for p in pieces]
    # Ignore "sentences" that contain no words at all (for example "...")
    return [s for s in sentences if tokenize_words(s)]


def moving_average_ttr(words, window=50):
    """
    Lexical diversity: the share of different words in every window of 50 words.
    Using a moving window makes long and short texts comparable.
    1.0 means no word is ever repeated; around 0.7 is typical for good writing.
    """
    words = [w.lower() for w in words]
    if not words:
        return 0.0
    if len(words) <= window:
        return len(set(words)) / len(words)
    ratios = [len(set(words[i:i + window])) / window for i in range(len(words) - window + 1)]
    return sum(ratios) / len(ratios)


def find_transitions(text):
    """Return {transition word: number of times used}."""
    lower = text.lower()
    found = {}
    for transition in TRANSITION_WORDS:
        count = len(re.findall(r"\b" + re.escape(transition) + r"\b", lower))
        if count:
            found[transition] = count
    return found


def content_words(words):
    """Meaningful words only: no stop words and no very short words."""
    return [w.lower() for w in words if w.lower() not in STOPWORDS and len(w) > 2]


def word_root(word):
    """A very simple way to match word families (education/educational/educate)."""
    word = word.lower()
    return word[:6] if len(word) > 6 else word


def analyze_text(text):
    """Run every measurement and return the results in one dictionary."""
    paragraphs = split_paragraphs(text)
    headings = [p for p in paragraphs if is_heading(p)]
    body_paragraphs = [p for p in paragraphs if not is_heading(p)] or paragraphs
    body_text = "\n\n".join(body_paragraphs)

    sentences = [s for p in body_paragraphs for s in split_sentences(p)]
    words = tokenize_words(body_text)
    sentence_lengths = [len(tokenize_words(s)) for s in sentences]
    meaningful = content_words(words)

    word_count = len(words)
    long_words = [w for w in words if len(w) >= 7]

    try:
        reading_ease = round(textstat.flesch_reading_ease(body_text), 1)
        grade_level = round(textstat.flesch_kincaid_grade(body_text), 1)
    except Exception:
        reading_ease, grade_level = None, None

    return {
        "text": text,
        "paragraphs": paragraphs,
        "body_paragraphs": body_paragraphs,
        "heading_count": len(headings),
        "sentences": sentences,
        "sentence_lengths": sentence_lengths,
        "words": words,
        "content_words": meaningful,
        "word_count": word_count,
        "unique_word_count": len({w.lower() for w in words}),
        "sentence_count": len(sentences),
        "paragraph_count": len(body_paragraphs),
        "avg_sentence_length": statistics.mean(sentence_lengths) if sentence_lengths else 0,
        "sentence_length_stdev": statistics.pstdev(sentence_lengths) if len(sentence_lengths) > 1 else 0,
        "avg_word_length": statistics.mean(len(w) for w in words) if words else 0,
        "long_word_ratio": len(long_words) / word_count if word_count else 0,
        "lexical_diversity": moving_average_ttr(words),
        "transitions": find_transitions(body_text),
        "top_keywords": Counter(meaningful).most_common(10),
        "reading_ease": reading_ease,
        "grade_level": grade_level,
        "reading_time_min": max(1, round(word_count / 200)),
    }
~~~~

## `utils/grammar_checker.py`

~~~~python
"""
grammar_checker.py
------------------
Finds grammar, spelling and punctuation problems using LanguageTool
(free and open source, no API key needed).

The checker tries three options, in this order:
  1. "local"  - LanguageTool running on your own computer (needs Java 17+).
               The first run downloads LanguageTool (about 250 MB) automatically.
  2. "online" - the free public LanguageTool server (needs internet, has rate limits).
  3. "basic"  - simple built-in rules, so the app still works if 1 and 2 fail.

Every problem is returned as a small dictionary:
  {"type": "Grammar" | "Spelling" | "Punctuation", "message": ..., "error_text": ...,
   "suggestions": [...], "context": ..., "rule": ...}
"""

import re

try:
    import language_tool_python
except ImportError:  # the app still works with the basic checker
    language_tool_python = None

MODE_DESCRIPTIONS = {
    "local": "LanguageTool (running on this computer)",
    "online": "LanguageTool (free public server)",
    "basic": "Basic built-in checks (LanguageTool unavailable)",
}

# LanguageTool rules that are about style or typography preferences, not errors
IGNORED_RULES = {"EN_QUOTES", "DASH_RULE", "ENGLISH_WORD_REPEAT_BEGINNING_RULE",
                 "PUNCTUATION_PARAGRAPH_END", "EN_UNPAIRED_QUOTES"}
IGNORED_ISSUE_TYPES = {"style", "locale-violation"}


class GrammarChecker:
    def __init__(self, language="en-US"):
        self.language = language
        self.tool = None
        self.mode = "basic"

        if language_tool_python is None:
            return
        try:
            self.tool = language_tool_python.LanguageTool(language)
            self.mode = "local"
        except Exception:
            try:
                self.tool = language_tool_python.LanguageToolPublicAPI(language)
                self.mode = "online"
            except Exception:
                self.tool = None
                self.mode = "basic"

    def check(self, text):
        """Return (list_of_issues, mode_used)."""
        if self.tool is not None:
            try:
                matches = self.tool.check(text)
                issues = [self._convert_match(m, text) for m in matches]
                return [i for i in issues if i is not None], self.mode
            except Exception:
                # e.g. no internet for the public server: use the basic rules instead
                pass
        return basic_check(text), "basic"

    @staticmethod
    def _convert_match(match, text):
        """Turn a LanguageTool match into our simple dictionary format."""
        # Newer versions use snake_case names, older ones use camelCase
        rule_id = getattr(match, "rule_id", None) or getattr(match, "ruleId", "") or ""
        issue_type = getattr(match, "rule_issue_type", None) or getattr(match, "ruleIssueType", "") or ""
        category = getattr(match, "category", "") or ""
        offset = getattr(match, "offset", 0) or 0
        length = getattr(match, "error_length", None) or getattr(match, "errorLength", 0) or 0

        if rule_id in IGNORED_RULES or issue_type in IGNORED_ISSUE_TYPES:
            return None

        if issue_type == "misspelling" or category == "TYPOS":
            kind = "Spelling"
        elif (category in ("PUNCTUATION", "TYPOGRAPHY", "CASING")
              or any(key in rule_id for key in ("PUNCT", "COMMA", "WHITESPACE"))):
            kind = "Punctuation"
        else:
            kind = "Grammar"

        return make_issue(text, offset, length, kind, getattr(match, "message", ""),
                          list(getattr(match, "replacements", []) or [])[:3], rule_id)


def make_issue(text, offset, length, kind, message, suggestions, rule):
    start, end = max(0, offset - 35), min(len(text), offset + length + 35)
    context = ("…" if start > 0 else "") + text[start:end].replace("\n", " ") + ("…" if end < len(text) else "")
    return {
        "type": kind,
        "message": message,
        "error_text": text[offset:offset + length],
        "suggestions": suggestions,
        "context": context,
        "rule": rule,
        "offset": offset,
        "length": length,
    }


# --------------------------------------------------------------------------
# Basic fallback checker (used only when LanguageTool cannot run)
# --------------------------------------------------------------------------

COMMON_MISSPELLINGS = {
    "recieve": "receive", "definately": "definitely", "seperate": "separate",
    "occured": "occurred", "untill": "until", "wich": "which", "becuase": "because",
    "beleive": "believe", "goverment": "government", "enviroment": "environment",
    "tommorow": "tomorrow", "accomodate": "accommodate", "alot": "a lot",
    "thier": "their", "truely": "truly", "wierd": "weird", "begining": "beginning",
    "arguement": "argument", "existance": "existence", "neccessary": "necessary",
    "knowlege": "knowledge", "freind": "friend", "calender": "calendar",
    "independant": "independent", "succesful": "successful", "teh": "the",
    "adn": "and", "writting": "writing", "grammer": "grammar", "comming": "coming",
    "acheive": "achieve", "beautifull": "beautiful", "carefull": "careful",
    "studing": "studying", "realy": "really", "finaly": "finally", "wether": "whether",
    "occurence": "occurrence", "responsability": "responsibility", "technolgy": "technology",
}

# (regular expression, type, message, suggestion)
BASIC_RULES = [
    (r"\b(\w+)\s+\1\b", "Grammar", "This word is repeated.", None),
    (r"(?<![\w.'’])i(?=[\s,;:!?])", "Grammar", "The pronoun 'I' should be a capital letter.", "I"),
    (r"\b[aA]\s+(?=(?!one\b|once\b)[aeioAEIO]\w)", "Grammar",
     "Use 'an' before a word that starts with a vowel sound.", "an "),
    (r"\b[aA]n\s+(?=[bcdfgjklmnpqrstvwxyzBCDFGJKLMNPQRSTVWXYZ]\w)", "Grammar",
     "Use 'a' before a word that starts with a consonant sound.", "a "),
    (r"[ \t]+[,.;:!?](?![\d.])", "Punctuation", "Remove the space before this punctuation mark.", None),
    (r"[,;](?=[A-Za-z])", "Punctuation", "Add a space after this punctuation mark.", None),
    (r"(?<=[a-z])\.(?=[A-Z][a-z])", "Punctuation", "Add a space after the full stop.", ". "),
    (r"(?<=\S) {2,}(?=\S)", "Punctuation", "There are extra spaces here.", " "),
    (r"[,;:]{2,}|(?<!\.)\.\.(?!\.)", "Punctuation", "Repeated punctuation mark.", None),
]


def basic_check(text):
    issues = []
    for pattern, kind, message, suggestion in BASIC_RULES:
        flags = re.IGNORECASE if pattern.startswith(r"\b(\w+)") else 0
        for m in re.finditer(pattern, text, flags):
            issues.append(make_issue(text, m.start(), m.end() - m.start(), kind, message,
                                     [suggestion] if suggestion else [], "BASIC_RULE"))

    # Sentences that start with a small letter
    for m in re.finditer(r"([.!?])\s+([a-z])", text):
        before = text[max(0, m.start() - 5):m.start() + 1].lower()
        if any(before.endswith(abbr) for abbr in ("e.g.", "i.e.", "etc.", "vs.", "a.m.", "p.m.")):
            continue
        if text[m.start(2):m.start(2) + 2] in ("i ", "i,"):
            continue  # already reported by the capital 'I' rule
        issues.append(make_issue(text, m.start(2), 1, "Punctuation",
                                 "A sentence should start with a capital letter.",
                                 [m.group(2).upper()], "BASIC_CAPITAL"))

    # Common spelling mistakes
    for m in re.finditer(r"[A-Za-z]+", text):
        correct = COMMON_MISSPELLINGS.get(m.group(0).lower())
        if correct:
            issues.append(make_issue(text, m.start(), len(m.group(0)), "Spelling",
                                     "Possible spelling mistake.", [correct], "BASIC_SPELLING"))
    return issues
~~~~

## `utils/scorer.py`

~~~~python
"""
scorer.py
---------
Turns the text analysis into a score out of 100.

    Grammar & Spelling    25 marks
    Vocabulary            20 marks
    Sentence Structure    20 marks
    Content & Relevance   25 marks
    Punctuation           10 marks

Each category function returns a dictionary with its score, a few details,
and the strengths / suggestions it found. All rules are simple, readable
"if this, then that" checks, so you can change them easily.

Remember: this is an automated ESTIMATE, not a human teacher's judgement.
"""

from collections import Counter

from .text_analyzer import analyze_text, tokenize_words, content_words, word_root

MIN_WORDS = 30
MAX_WORDS = 5000

MAX_SCORES = {
    "Grammar & Spelling": 25,
    "Vocabulary": 20,
    "Sentence Structure": 20,
    "Content & Relevance": 25,
    "Punctuation": 10,
}

REMARKS = [  # (minimum score, remark, short description)
    (90, "Excellent", "Outstanding writing with very few weaknesses."),
    (75, "Very Good", "Strong writing with only minor areas to polish."),
    (60, "Good", "Solid writing with some clear areas to improve."),
    (45, "Average", "Understandable writing that needs work in several areas."),
    (0, "Needs Improvement", "The writing needs significant revision and practice."),
]


class EvaluationError(Exception):
    """Raised when the text cannot be evaluated (too short, too long)."""


def scale(value, low, high, points):
    """
    Map a value onto 0..points in a straight line.
    value <= low gives 0 points, value >= high gives full points.
    (If low is bigger than high, smaller values are better.)
    """
    if high == low:
        return points
    ratio = (value - low) / (high - low)
    return max(0.0, min(1.0, ratio)) * points


def result(score, max_score, details, strengths, suggestions):
    return {"score": round(score, 1), "max": max_score, "details": details,
            "strengths": strengths, "suggestions": suggestions}


# ---------------------------------------------------------------------------
# 1. Grammar & Spelling (25)
# ---------------------------------------------------------------------------
def score_grammar(analysis, issues):
    grammar = [i for i in issues if i["type"] == "Grammar"]
    spelling = [i for i in issues if i["type"] == "Spelling"]
    errors = len(grammar) + len(spelling)
    rate = errors / analysis["word_count"] * 100  # errors per 100 words

    # 0 errors per 100 words = 25 marks, 8 or more errors per 100 words = 0 marks
    score = scale(rate, 8, 0, 25)

    strengths, suggestions = [], []
    if errors == 0:
        strengths.append("No grammar or spelling mistakes were detected.")
    elif rate < 1.5:
        strengths.append(f"Very few grammar and spelling mistakes (only {errors} found).")

    if spelling:
        unique_words = list(dict.fromkeys(i["error_text"] for i in spelling))
        examples = ", ".join(f"'{w}'" for w in unique_words[:3])
        suggestions.append(f"Correct {len(spelling)} possible spelling mistake(s), such as {examples}.")
    if grammar and rate >= 1.5:
        common = Counter(i["message"] for i in grammar).most_common(1)[0][0]
        suggestions.append(f"Proofread for grammar ({len(grammar)} issue(s) found). "
                           f"Most common: \"{common}\"")

    details = {"Grammar issues": len(grammar), "Spelling issues": len(spelling),
               "Errors per 100 words": round(rate, 1)}
    return result(score, 25, details, strengths, suggestions)


# ---------------------------------------------------------------------------
# 2. Vocabulary (20)
# ---------------------------------------------------------------------------
def score_vocabulary(analysis, topic_roots):
    diversity = analysis["lexical_diversity"]
    long_ratio = analysis["long_word_ratio"]

    diversity_points = scale(diversity, 0.55, 0.80, 10)          # variety of words
    sophistication_points = scale(long_ratio, 0.08, 0.25, 6)      # longer, richer words

    # Words used too often (topic words are allowed to repeat)
    limit = max(4, analysis["word_count"] * 0.025)
    overused = [(w, c) for w, c in Counter(analysis["content_words"]).most_common(10)
                if c > limit and word_root(w) not in topic_roots]
    repetition_points = max(0, 4 - len(overused))

    score = diversity_points + sophistication_points + repetition_points

    strengths, suggestions = [], []
    if diversity >= 0.72:
        strengths.append("Uses a wide variety of words without much repetition.")
    elif diversity < 0.62:
        suggestions.append("Vary your word choice. Use synonyms instead of repeating the same words.")
    if long_ratio >= 0.2:
        strengths.append("Good use of rich, descriptive vocabulary.")
    elif long_ratio < 0.12:
        suggestions.append("Try using more precise or advanced words where they fit naturally "
                           "(for example 'significant' instead of 'big').")
    if overused:
        words = ", ".join(f"'{w}' ({c} times)" for w, c in overused[:4])
        suggestions.append(f"Some words are used very often: {words}. Replace a few with synonyms.")

    details = {"Lexical diversity (0–1)": round(diversity, 2),
               "Unique words": analysis["unique_word_count"],
               "Long words (7+ letters)": f"{long_ratio:.0%}",
               "Average word length": round(analysis["avg_word_length"], 1)}
    return result(score, 20, details, strengths, suggestions)


# ---------------------------------------------------------------------------
# 3. Sentence Structure (20)
# ---------------------------------------------------------------------------
def score_sentences(analysis):
    lengths = analysis["sentence_lengths"]
    count = len(lengths)
    avg = analysis["avg_sentence_length"]
    stdev = analysis["sentence_length_stdev"]

    # (a) Average length: 12–22 words is a comfortable range (8 marks)
    if 12 <= avg <= 22:
        length_points = 8
    elif avg < 12:
        length_points = scale(avg, 5, 12, 8)
    else:
        length_points = scale(avg, 40, 22, 8)

    # (b) Variety: mix of short and long sentences + different openings (6 marks)
    variety_points = scale(stdev, 2, 8, 4)
    first_words = [tokenize_words(s)[0].lower() for s in analysis["sentences"] if tokenize_words(s)]
    starter_variety = len(set(first_words)) / len(first_words) if first_words else 1
    variety_points += 2 if count < 3 else scale(starter_variety, 0.4, 0.8, 2)

    # (c) Problem sentences: very long (run-on) or very short (fragment) (6 marks)
    long_sentences = [n for n in lengths if n > 40]
    short_sentences = [n for n in lengths if n < 4]
    problem_ratio = (len(long_sentences) + len(short_sentences)) / count if count else 1
    problem_points = 6 * max(0.0, 1 - problem_ratio * 2.5)

    score = length_points + variety_points + problem_points

    strengths, suggestions = [], []
    if 12 <= avg <= 22:
        strengths.append(f"Sentences have a comfortable average length ({avg:.0f} words).")
    elif avg < 12:
        suggestions.append("Many sentences are short. Join related ideas with words like "
                           "'and', 'because', 'although' or 'which'.")
    else:
        suggestions.append(f"Sentences are long on average ({avg:.0f} words). "
                           "Split some of them so they are easier to read.")
    if stdev >= 6:
        strengths.append("Good mix of short and long sentences keeps the writing interesting.")
    elif count >= 4 and stdev < 3.5:
        suggestions.append("Most sentences are about the same length. Mix short and long sentences.")
    if long_sentences:
        suggestions.append(f"{len(long_sentences)} sentence(s) are over 40 words long. "
                           "These may be run-on sentences; consider splitting them.")
    if short_sentences:
        suggestions.append(f"{len(short_sentences)} sentence(s) have fewer than 4 words. "
                           "Check that they are complete sentences.")
    if first_words and count >= 5:
        word, times = Counter(first_words).most_common(1)[0]
        if times >= 3 and times / count > 0.25:
            suggestions.append(f"{times} sentences begin with '{word.capitalize()}'. "
                               "Vary how your sentences start.")

    details = {"Sentences": count,
               "Average sentence length": f"{avg:.1f} words",
               "Longest sentence": f"{max(lengths) if lengths else 0} words",
               "Shortest sentence": f"{min(lengths) if lengths else 0} words"}
    return result(score, 20, details, strengths, suggestions)


# ---------------------------------------------------------------------------
# 4. Content & Relevance (25)
# ---------------------------------------------------------------------------
def score_content(analysis, topic, topic_keywords):
    words = analysis["word_count"]
    paragraphs = analysis["paragraph_count"]
    transitions = analysis["transitions"]

    # (a) Length / development of ideas (8 marks)
    length_points = scale(words, 60, 300, 8)

    # (b) Organisation into paragraphs (5 marks)
    if paragraphs >= 4:
        organisation_points = 5
    elif paragraphs == 1 and words < 150:
        organisation_points = 3.5  # a short piece can reasonably be one paragraph
    else:
        organisation_points = {1: 1.5, 2: 3, 3: 4.5}[paragraphs]

    # (c) Linking words / flow between ideas (5 marks)
    target = min(8, 3 + words // 150)
    transition_points = scale(len(transitions), 0, target, 5)

    # (d) Relevance / focus (7 marks)
    strengths, suggestions = [], []
    # The title counts too, so use the whole text here
    essay_roots = Counter(word_root(w) for w in content_words(tokenize_words(analysis["text"])))
    if topic_keywords:
        found = [k for k in topic_keywords if word_root(k) in essay_roots]
        missing = [k for k in topic_keywords if k not in found]
        coverage = len(found) / len(topic_keywords)
        mentions = sum(essay_roots[word_root(k)] for k in topic_keywords)
        density = mentions / words * 100
        relevance_points = scale(coverage, 0.2, 0.8, 5) + scale(density, 0.5, 2.5, 2)
        if coverage >= 0.7:
            strengths.append("The writing stays closely connected to the given topic.")
        else:
            suggestions.append("Connect the writing more clearly to the topic. Key ideas not "
                               f"mentioned: {', '.join(missing[:5])}.")
        relevance_detail = f"{coverage:.0%} of topic keywords used"
    else:
        # No topic given: check that the main keywords appear across the paragraphs
        top = [w for w, _ in analysis["top_keywords"][:3]]
        body = analysis["body_paragraphs"]
        covered = sum(1 for p in body if any(word_root(t) in {word_root(w) for w in content_words(tokenize_words(p))} for t in top))
        paragraph_coverage = covered / len(body) if body else 0
        relevance_points = 5 + scale(paragraph_coverage, 0.5, 1.0, 2)
        relevance_detail = "Estimated (no topic given)"

    score = length_points + organisation_points + transition_points + relevance_points

    if words >= 250:
        strengths.append(f"Ideas are well developed ({words} words).")
    elif words < 150:
        suggestions.append(f"The writing is quite short ({words} words). Add explanations, "
                           "examples or evidence to develop your ideas.")
    if paragraphs >= 3:
        strengths.append(f"Clearly organised into {paragraphs} paragraphs.")
    elif words >= 150:
        suggestions.append("Organise the writing into paragraphs: an introduction, "
                           "body paragraphs and a conclusion.")
    if len(transitions) >= target:
        strengths.append("Good use of linking words (e.g. " +
                         ", ".join(f"'{t}'" for t in list(transitions)[:3]) + ") to connect ideas.")
    else:
        suggestions.append("Use more linking words such as 'however', 'for example', "
                           "'therefore' and 'in conclusion' to connect your ideas.")

    details = {"Word count": words, "Paragraphs": paragraphs,
               "Linking words used": len(transitions), "Topic relevance": relevance_detail}
    return result(score, 25, details, strengths, suggestions)


# ---------------------------------------------------------------------------
# 5. Punctuation (10)
# ---------------------------------------------------------------------------
def score_punctuation(analysis, issues):
    text = "\n\n".join(analysis["body_paragraphs"])
    words = analysis["word_count"]
    sentences = max(1, analysis["sentence_count"])
    punctuation_issues = [i for i in issues if i["type"] == "Punctuation"]

    # (a) Punctuation & capitalisation errors found by the checker (5 marks)
    rate = len(punctuation_issues) / words * 100
    issue_points = scale(rate, 4, 0, 5)

    # (b) Every paragraph ends with . ! or ? (2 marks)
    body = analysis["body_paragraphs"]
    missing_end = sum(1 for p in body if not p.rstrip("\"'”’)]").endswith((".", "!", "?")))
    end_points = 2 * (1 - missing_end / len(body)) if body else 0

    # (c) Commas used in longer sentences (2 marks)
    commas_per_sentence = text.count(",") / sentences
    if analysis["avg_sentence_length"] < 12:
        comma_points = 2
    else:
        comma_points = scale(commas_per_sentence, 0.1, 0.5, 2)

    # (d) Exclamation marks not overused (1 mark)
    exclamations = text.count("!")
    exclamation_points = 0 if exclamations / sentences > 0.3 else 1

    score = issue_points + end_points + comma_points + exclamation_points

    strengths, suggestions = [], []
    if not punctuation_issues and not missing_end:
        strengths.append("Punctuation and capitalisation are used correctly.")
    if punctuation_issues:
        suggestions.append(f"Fix {len(punctuation_issues)} punctuation or capitalisation issue(s) "
                           "(see the Issues tab).")
    if missing_end:
        suggestions.append(f"{missing_end} paragraph(s) do not end with a full stop, "
                           "question mark or exclamation mark.")
    if comma_points < 1:
        suggestions.append("Use commas to separate clauses and items in longer sentences.")
    if exclamation_points == 0:
        suggestions.append("Exclamation marks are overused. Save them for real emphasis.")

    details = {"Punctuation issues": len(punctuation_issues),
               "Commas per sentence": round(commas_per_sentence, 2),
               "Paragraphs missing end mark": missing_end,
               "Exclamation marks": exclamations}
    return result(score, 10, details, strengths, suggestions)


# ---------------------------------------------------------------------------
# Putting it all together
# ---------------------------------------------------------------------------
def get_remark(total):
    for minimum, remark, description in REMARKS:
        if total >= minimum:
            return remark, description
    return REMARKS[-1][1], REMARKS[-1][2]


def evaluate_writing(text, checker, topic=""):
    """Analyse and score the text. Returns one dictionary with everything."""
    analysis = analyze_text(text)

    if analysis["word_count"] < MIN_WORDS:
        raise EvaluationError(f"The text has only {analysis['word_count']} words. "
                              f"Please upload at least {MIN_WORDS} words for a meaningful evaluation.")
    if analysis["word_count"] > MAX_WORDS:
        raise EvaluationError(f"The text has {analysis['word_count']:,} words. "
                              f"Please upload a piece of writing under {MAX_WORDS:,} words.")

    issues, checker_mode = checker.check(text)

    topic_keywords = list(dict.fromkeys(content_words(tokenize_words(topic or ""))))
    topic_roots = {word_root(k) for k in topic_keywords}

    categories = {
        "Grammar & Spelling": score_grammar(analysis, issues),
        "Vocabulary": score_vocabulary(analysis, topic_roots),
        "Sentence Structure": score_sentences(analysis),
        "Content & Relevance": score_content(analysis, topic, topic_keywords),
        "Punctuation": score_punctuation(analysis, issues),
    }

    total = round(sum(c["score"] for c in categories.values()), 1)
    remark, remark_description = get_remark(total)

    percentages = {name: c["score"] / c["max"] for name, c in categories.items()}
    best = max(percentages, key=percentages.get)
    weakest = min(percentages, key=percentages.get)
    overall_comment = (f"{remark_description} The strongest area is {best} "
                       f"({percentages[best]:.0%}), and the area that needs the most attention "
                       f"is {weakest} ({percentages[weakest]:.0%}).")

    strengths = [s for c in categories.values() for s in c["strengths"]]
    # Show suggestions for the weakest categories first
    suggestions = [s for name in sorted(percentages, key=percentages.get)
                   for s in categories[name]["suggestions"]]
    if not strengths:
        strengths = ["The writing was submitted and analysed. Keep practising to build strengths."]
    if not suggestions:
        suggestions = ["Keep up the good work. Ask a teacher or peer to review it for deeper feedback."]

    return {
        "total": total,
        "remark": remark,
        "overall_comment": overall_comment,
        "categories": categories,
        "strengths": strengths,
        "suggestions": suggestions,
        "issues": issues,
        "checker_mode": checker_mode,
        "topic": topic.strip() if topic else "",
        "stats": {
            "Words": analysis["word_count"],
            "Sentences": analysis["sentence_count"],
            "Paragraphs": analysis["paragraph_count"],
            "Reading time": f"{analysis['reading_time_min']} min",
            "Reading ease (0–100)": analysis["reading_ease"],
            "Grade level": analysis["grade_level"],
        },
        "top_keywords": analysis["top_keywords"],
        "transitions": analysis["transitions"],
        "sentence_lengths": analysis["sentence_lengths"],
    }
~~~~

## `utils/report_generator.py`

~~~~python
"""
report_generator.py
-------------------
Builds the downloadable evaluation report in two formats:
  * a plain text (.txt) report
  * a Word (.docx) report
"""

import io
from datetime import datetime

from docx import Document
from docx.shared import Pt, RGBColor

DISCLAIMER = ("This score is an automated estimate produced by a computer program. "
              "It is not a perfect human evaluation. It cannot fully judge creativity, "
              "argument quality, tone or accuracy of ideas, so please use it as a guide "
              "alongside feedback from a teacher or reviewer.")


def build_text_report(evaluation, file_name):
    line = "=" * 60
    parts = [
        line,
        "AUTOMATED WRITING EVALUATION REPORT",
        line,
        f"File:      {file_name}",
        f"Date:      {datetime.now().strftime('%d %B %Y, %I:%M %p')}",
        f"Topic:     {evaluation['topic'] or 'Not provided'}",
        f"Checker:   {evaluation['checker_mode']}",
        "",
        f"TOTAL SCORE: {evaluation['total']} / 100",
        f"REMARK:      {evaluation['remark']}",
        "",
        evaluation["overall_comment"],
        "",
        "-" * 60,
        "CATEGORY SCORES",
        "-" * 60,
    ]
    for name, cat in evaluation["categories"].items():
        parts.append(f"{name:<24}{cat['score']:>5} / {cat['max']}")
        for key, value in cat["details"].items():
            parts.append(f"    - {key}: {value}")

    parts += ["", "-" * 60, "TEXT STATISTICS", "-" * 60]
    parts += [f"{key}: {value}" for key, value in evaluation["stats"].items()]

    parts += ["", "-" * 60, "STRENGTHS", "-" * 60]
    parts += [f"+ {s}" for s in evaluation["strengths"]]

    parts += ["", "-" * 60, "SUGGESTIONS FOR IMPROVEMENT", "-" * 60]
    parts += [f"* {s}" for s in evaluation["suggestions"]]

    if evaluation["issues"]:
        parts += ["", "-" * 60, f"ISSUES FOUND ({len(evaluation['issues'])})", "-" * 60]
        for n, issue in enumerate(evaluation["issues"], 1):
            fix = f" -> suggestion: {', '.join(issue['suggestions'])}" if issue["suggestions"] else ""
            parts.append(f"{n}. [{issue['type']}] '{issue['error_text']}': {issue['message']}{fix}")

    parts += ["", line, "IMPORTANT NOTE", line, DISCLAIMER, ""]
    return "\n".join(parts)


def build_docx_report(evaluation, file_name):
    doc = Document()
    doc.styles["Normal"].font.name = "Calibri"
    doc.styles["Normal"].font.size = Pt(11)

    doc.add_heading("Automated Writing Evaluation Report", level=0)
    info = doc.add_paragraph()
    info.add_run(f"File: {file_name}\n")
    info.add_run(f"Date: {datetime.now().strftime('%d %B %Y, %I:%M %p')}\n")
    info.add_run(f"Topic: {evaluation['topic'] or 'Not provided'}")

    score = doc.add_paragraph()
    run = score.add_run(f"Total score: {evaluation['total']} / 100  ({evaluation['remark']})")
    run.bold = True
    run.font.size = Pt(16)
    run.font.color.rgb = RGBColor(0xB3, 0x26, 0x1E)
    doc.add_paragraph(evaluation["overall_comment"])

    doc.add_heading("Category scores", level=1)
    table = doc.add_table(rows=1, cols=3)
    table.style = "Light Grid Accent 1"
    header = table.rows[0].cells
    header[0].text, header[1].text, header[2].text = "Category", "Score", "Out of"
    for name, cat in evaluation["categories"].items():
        row = table.add_row().cells
        row[0].text, row[1].text, row[2].text = name, str(cat["score"]), str(cat["max"])

    doc.add_heading("Text statistics", level=1)
    for key, value in evaluation["stats"].items():
        doc.add_paragraph(f"{key}: {value}", style="List Bullet")

    doc.add_heading("Strengths", level=1)
    for s in evaluation["strengths"]:
        doc.add_paragraph(s, style="List Bullet")

    doc.add_heading("Suggestions for improvement", level=1)
    for s in evaluation["suggestions"]:
        doc.add_paragraph(s, style="List Bullet")

    if evaluation["issues"]:
        doc.add_heading(f"Issues found ({len(evaluation['issues'])})", level=1)
        for issue in evaluation["issues"][:100]:
            fix = f" Suggestion: {', '.join(issue['suggestions'])}." if issue["suggestions"] else ""
            doc.add_paragraph(f"[{issue['type']}] \"{issue['error_text']}\": {issue['message']}{fix}",
                              style="List Number")

    doc.add_heading("Important note", level=1)
    note = doc.add_paragraph(DISCLAIMER)
    note.runs[0].italic = True

    buffer = io.BytesIO()
    doc.save(buffer)
    return buffer.getvalue()
~~~~

## `app.py`

~~~~python
"""
app.py
------
Automated Writing Scoring and Evaluation System (Streamlit web app).

Run it with:   streamlit run app.py
"""

import hashlib
import html

import pandas as pd
import streamlit as st

from utils.file_reader import read_uploaded_file, FileReadError, MAX_FILE_SIZE_MB
from utils.grammar_checker import GrammarChecker, MODE_DESCRIPTIONS
from utils.scorer import evaluate_writing, EvaluationError, MAX_SCORES, REMARKS, MIN_WORDS
from utils.report_generator import build_text_report, build_docx_report, DISCLAIMER

st.set_page_config(page_title="Writing Evaluator", page_icon="✍️", layout="wide")

REMARK_COLORS = {"Excellent": "#1E7B4F", "Very Good": "#2E6A9E", "Good": "#7A6A12",
                 "Average": "#B0601A", "Needs Improvement": "#B3261E"}

# ---------------------------------------------------------------------------
# Styling: ink-blue text with a teacher's red-pen score
# ---------------------------------------------------------------------------
st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Caveat:wght@600;700&family=Literata:opsz,wght@7..72,500;7..72,700&display=swap');

h1, h2, h3, .serif { font-family: 'Literata', Georgia, serif !important; color: #1F3A5F; }
.block-container { padding-top: 2.2rem; max-width: 1150px; }

.hero h1 { font-size: 2.3rem; margin-bottom: .2rem; }
.hero p { color: #4A5568; font-size: 1.05rem; margin-top: 0; max-width: 62ch; }

/* The marked paper with the red-pen score */
.paper { background: #FFFFFF; border: 1px solid #E2E6EC; border-radius: 6px;
         padding: 1.6rem 1.8rem; display: flex; gap: 2rem; align-items: center;
         background-image: repeating-linear-gradient(#FFFFFF 0 31px, #E8EEF6 31px 32px); }
.grade { flex: 0 0 auto; text-align: center; transform: rotate(-4deg); }
.grade .ring { font-family: 'Caveat', 'Segoe Print', 'Bradley Hand', cursive; color: #B3261E; font-size: 4.6rem; line-height: 1;
               border: 3px solid #B3261E; border-radius: 52% 46% 55% 44% / 50% 55% 45% 52%;
               padding: .55rem 1.3rem .35rem; display: inline-block; background: #FFFFFFCC; }
.grade .ring small { font-size: 1.8rem; }
.grade .remark { font-family: 'Caveat', 'Segoe Print', 'Bradley Hand', cursive; font-size: 1.9rem; margin-top: .3rem; }
.paper .comment { font-size: 1.02rem; color: #1C2430; line-height: 1.6; }
.paper .stats { color: #4A5568; font-size: .92rem; margin-top: .6rem; }

/* Rubric rows */
.rubric-row { display: grid; grid-template-columns: 190px 1fr 80px; gap: 1rem;
              align-items: center; padding: .55rem 0; border-bottom: 1px dashed #D5DCE5; }
.rubric-row .name { font-weight: 600; color: #1F3A5F; }
.bar { height: 10px; background: #E8EEF6; border-radius: 5px; overflow: hidden; }
.bar span { display: block; height: 100%; border-radius: 5px; }
.rubric-row .marks { text-align: right; font-variant-numeric: tabular-nums; font-weight: 600; }

.note { background: #FFF7E6; border-left: 4px solid #D9A21B; padding: .8rem 1rem;
        border-radius: 4px; color: #4A3B12; font-size: .93rem; }
.feedback li { margin-bottom: .45rem; line-height: 1.5; }
mark.issue { background: #FDE2E0; border-bottom: 2px solid #B3261E; padding: 0 2px; }
mark.issue.Spelling { background: #FFF0C2; border-bottom-color: #C58A00; }
mark.issue.Punctuation { background: #E3ECFA; border-bottom-color: #2E6A9E; }
.essay { background: #fff; border: 1px solid #E2E6EC; border-radius: 6px; padding: 1.2rem 1.4rem;
         line-height: 1.8; white-space: pre-wrap; font-family: 'Literata', Georgia, serif; }

@media (max-width: 700px) {
  .paper { flex-direction: column; text-align: center; }
  .rubric-row { grid-template-columns: 1fr 70px; }
  .rubric-row .bar { grid-column: 1 / -1; grid-row: 2; }
}
</style>
""", unsafe_allow_html=True)


# ---------------------------------------------------------------------------
# Cached helpers (so LanguageTool starts only once, and each file is scored once)
# ---------------------------------------------------------------------------
@st.cache_resource(show_spinner=False)
def get_checker(language):
    return GrammarChecker(language)


@st.cache_data(show_spinner=False)
def run_evaluation(text, topic, language):
    return evaluate_writing(text, get_checker(language), topic)


def highlight_issues(text, issues):
    """Return the text as HTML with each issue wrapped in a coloured <mark>."""
    pieces, position = [], 0
    for issue in sorted(issues, key=lambda i: i["offset"]):
        start, end = issue["offset"], issue["offset"] + issue["length"]
        if start < position or end <= start:
            continue  # skip overlapping issues
        tip = html.escape(issue["message"], quote=True)
        pieces.append(html.escape(text[position:start]))
        pieces.append(f'<mark class="issue {issue["type"]}" title="{tip}">'
                      f'{html.escape(text[start:end])}</mark>')
        position = end
    pieces.append(html.escape(text[position:]))
    return "".join(pieces)


# ---------------------------------------------------------------------------
# Sidebar
# ---------------------------------------------------------------------------
with st.sidebar:
    st.markdown("### Settings")
    language_label = st.selectbox("English spelling", ["British English (en-GB)", "American English (en-US)"],
                                  help="Choose which spelling rules to use, e.g. 'colour' vs 'color'.")
    language = "en-GB" if "GB" in language_label else "en-US"

    st.markdown("### How the score is made up")
    st.table(pd.DataFrame({"Category": list(MAX_SCORES), "Marks": list(MAX_SCORES.values())})
             .set_index("Category"))

    st.markdown("### Remarks")
    st.markdown("\n".join(f"- **{r}**: {m}+" if m else f"- **{r}**: below 45" for m, r, _ in REMARKS))

    st.markdown("### About")
    st.caption("Built with Python, Streamlit, LanguageTool and simple NLP. "
               "No paid APIs or API keys are used.")


# ---------------------------------------------------------------------------
# Header and upload
# ---------------------------------------------------------------------------
st.markdown("""
<div class="hero">
  <h1>Automated Writing Evaluator</h1>
  <p>Upload an essay, story or report. It is marked out of 100 for grammar, vocabulary,
  sentence structure, content and punctuation, with feedback on what to improve.</p>
</div>
""", unsafe_allow_html=True)

col_upload, col_topic = st.columns([3, 2])
with col_upload:
    uploaded_file = st.file_uploader(f"Upload your writing (TXT or DOCX, up to {MAX_FILE_SIZE_MB} MB)",
                                     type=["txt", "docx"])
with col_topic:
    topic = st.text_input("Topic or question (optional)",
                          placeholder="e.g. The impact of social media on students",
                          help="If you add the topic, the relevance check is more accurate.")

if uploaded_file is None:
    st.info(f"Choose a file to start. Scoring begins automatically once it is uploaded. "
            f"The writing should be at least {MIN_WORDS} words long.")
    st.markdown(f'<div class="note"><b>Please note:</b> {DISCLAIMER}</div>', unsafe_allow_html=True)
    st.stop()

# ---------------------------------------------------------------------------
# Read and evaluate (runs automatically after upload)
# ---------------------------------------------------------------------------
try:
    text = read_uploaded_file(uploaded_file)
except FileReadError as error:
    st.error(f"**Cannot read this file.** {error}")
    st.stop()

first_run = "checker_ready" not in st.session_state
spinner_text = ("Starting the grammar checker. The first run downloads LanguageTool, "
                "which can take a few minutes…" if first_run else "Evaluating your writing…")
try:
    with st.spinner(spinner_text):
        evaluation = run_evaluation(text, topic.strip(), language)
    st.session_state["checker_ready"] = True
except EvaluationError as error:
    st.warning(f"**This text can’t be scored yet.** {error}")
    st.stop()
except Exception as error:  # anything unexpected: show it instead of crashing
    st.error(f"Something went wrong while evaluating the file: {error}")
    st.stop()

if evaluation["checker_mode"] == "basic":
    st.warning("LanguageTool could not be started, so only basic grammar and spelling checks were used. "
               "Install Java 17 or newer (or connect to the internet) for full checking. "
               "See the README for help.")

# ---------------------------------------------------------------------------
# Results: total score and rubric
# ---------------------------------------------------------------------------
color = REMARK_COLORS[evaluation["remark"]]
stats = evaluation["stats"]
total = evaluation["total"]
total_text = f"{total:.0f}" if total == int(total) else f"{total:.1f}"

st.markdown(f"""
<div class="paper">
  <div class="grade">
    <div class="ring">{total_text}<small>/100</small></div>
    <div class="remark" style="color:{color}">{evaluation['remark']}</div>
  </div>
  <div>
    <div class="comment">{html.escape(evaluation['overall_comment'])}</div>
    <div class="stats">{stats['Words']} words, {stats['Sentences']} sentences,
      {stats['Paragraphs']} paragraph(s), about {stats['Reading time']} to read.
      File: {html.escape(uploaded_file.name)}</div>
  </div>
</div>
""", unsafe_allow_html=True)

st.markdown("### Marks by category")
rows = []
for name, cat in evaluation["categories"].items():
    percent = cat["score"] / cat["max"] * 100
    bar_color = "#1E7B4F" if percent >= 75 else "#2E6A9E" if percent >= 60 else "#B0601A" if percent >= 45 else "#B3261E"
    rows.append(f"""<div class="rubric-row"><div class="name">{name}</div>
        <div class="bar"><span style="width:{percent:.0f}%;background:{bar_color}"></span></div>
        <div class="marks">{cat['score']} / {cat['max']}</div></div>""")
st.markdown("".join(rows), unsafe_allow_html=True)
st.write("")

# ---------------------------------------------------------------------------
# Tabs with feedback and details
# ---------------------------------------------------------------------------
issue_count = len(evaluation["issues"])
tab_feedback, tab_details, tab_issues, tab_text = st.tabs(
    ["Feedback", "Detailed analysis", f"Issues found ({issue_count})", "Your text"])

with tab_feedback:
    left, right = st.columns(2)
    with left:
        st.markdown("#### Strengths")
        st.markdown('<ul class="feedback">' + "".join(f"<li>{html.escape(s)}</li>" for s in evaluation["strengths"])
                    + "</ul>", unsafe_allow_html=True)
    with right:
        st.markdown("#### Suggestions for improvement")
        st.markdown('<ul class="feedback">' + "".join(f"<li>{html.escape(s)}</li>" for s in evaluation["suggestions"])
                    + "</ul>", unsafe_allow_html=True)

with tab_details:
    for name, cat in evaluation["categories"].items():
        with st.expander(f"{name}: {cat['score']} / {cat['max']}"):
            st.table(pd.DataFrame({"Measure": list(cat["details"]),
                                   "Value": [str(v) for v in cat["details"].values()]}).set_index("Measure"))

    col_a, col_b = st.columns(2)
    with col_a:
        st.markdown("#### Text statistics")
        st.table(pd.DataFrame({"Statistic": list(stats), "Value": [str(v) for v in stats.values()]})
                 .set_index("Statistic"))
        st.caption("Reading ease: higher means easier to read (60–70 is plain English). "
                   "Grade level: the school grade needed to understand the text.")
    with col_b:
        st.markdown("#### Main keywords")
        keywords = evaluation["top_keywords"]
        if keywords:
            st.bar_chart(pd.DataFrame(keywords, columns=["Keyword", "Times used"]).set_index("Keyword"))
        if evaluation["transitions"]:
            st.markdown("**Linking words used:** " + ", ".join(
                f"{w} ({n})" for w, n in evaluation["transitions"].items()))

    st.markdown("#### Sentence lengths")
    lengths = evaluation["sentence_lengths"]
    st.bar_chart(pd.DataFrame({"Words in sentence": lengths}, index=range(1, len(lengths) + 1)))
    st.caption("Each bar is one sentence, in order. A good mix of heights shows varied sentence structure.")

with tab_issues:
    st.caption(f"Checked with: {MODE_DESCRIPTIONS[evaluation['checker_mode']]}")
    if not evaluation["issues"]:
        st.success("No grammar, spelling or punctuation issues were found.")
    else:
        types = st.multiselect("Show", ["Grammar", "Spelling", "Punctuation"],
                               default=["Grammar", "Spelling", "Punctuation"])
        table = pd.DataFrame([{
            "Type": i["type"],
            "Text": i["error_text"],
            "Problem": i["message"],
            "Suggestion": ", ".join(i["suggestions"]) or "-",
            "Context": i["context"],
        } for i in evaluation["issues"] if i["type"] in types])
        st.dataframe(table, width="stretch", hide_index=True)

with tab_text:
    st.caption("Highlighted: red = grammar, yellow = spelling, blue = punctuation. "
               "Hover over a highlight to see the problem.")
    st.markdown(f'<div class="essay">{highlight_issues(text, evaluation["issues"])}</div>',
                unsafe_allow_html=True)

# ---------------------------------------------------------------------------
# Download the report
# ---------------------------------------------------------------------------
st.markdown("### Download the report")
base_name = uploaded_file.name.rsplit(".", 1)[0]
key = hashlib.md5((text + topic).encode()).hexdigest()[:8]
d1, d2, _ = st.columns([1, 1, 2])
with d1:
    st.download_button("Download report (TXT)", build_text_report(evaluation, uploaded_file.name),
                       file_name=f"{base_name}_evaluation.txt", mime="text/plain",
                       width="stretch", key=f"txt_{key}")
with d2:
    st.download_button("Download report (Word)", build_docx_report(evaluation, uploaded_file.name),
                       file_name=f"{base_name}_evaluation.docx",
                       mime="application/vnd.openxmlformats-officedocument.wordprocessingml.document",
                       width="stretch", key=f"docx_{key}")

st.write("")
st.markdown(f'<div class="note"><b>Please note:</b> {DISCLAIMER}</div>', unsafe_allow_html=True)
~~~~

## `sample_files/sample_essay.txt`

~~~~text
The Impact of Technology on Education

Technology has changed the way students learn in many important ways. In the past, students depended almost entirely on textbooks and classroom lectures. Today, however, learners can access online courses, educational videos and digital libraries from almost anywhere in the world.

One major benefit of technology is that it makes learning more flexible. For example, a student who misses a lesson can watch a recorded lecture later in the evening. Interactive tools, such as quizzes and simulations, also help students understand difficult concepts by letting them practise at their own pace. As a result, many learners feel more confident and motivated.

On the other hand, technology also creates some serious challenges. Many students are easily distracted by social media and games while they are studying. Furthermore, not every family can afford a laptop or a reliable internet connection, which means that some children are left behind. Teachers must therefore find ways to use technology wisely and fairly.

In conclusion, technology is a powerful tool that can improve education when it is used carefully. Schools, parents and governments should work together to make sure that every student benefits from it. If they succeed, the classrooms of the future will be more inclusive, engaging and effective than ever before.
~~~~

## `sample_files/sample_with_mistakes.txt`

~~~~text
my favourite holiday

last summer i go to the beach with my family. it was very fun. we play in the water and we eat alot of ice cream. the weather was very hot and very sunny. i like the beach very much because the beach is very nice.

My brother and me builded a big sand castle , it was very big. Then the the waves came and destroy it. we was very sad but then we laughed. My mom take many photo of us.

i think this was the best holiday ever!!! I want to go there again next year becuase it was very very fun
~~~~

## `README.md`

~~~~markdown
# Automated Writing Scoring and Evaluation System

A beginner-friendly Python web app that reads a piece of writing (TXT or DOCX),
analyses it automatically and gives it a score out of 100, with remarks,
strengths and suggestions for improvement.

> **Important:** the score is an **automated estimate**, not a perfect human evaluation.
> A computer can count mistakes and measure sentence lengths, but it cannot fully judge
> creativity, the quality of an argument, tone, or whether the facts are correct.
> Use the result as a helpful guide alongside feedback from a teacher or reviewer.

---

## 1. What the project does (in simple words)

1. You open the web page and upload a `.txt` or `.docx` file.
2. The app pulls the text out of the file.
3. It checks grammar, spelling and punctuation with **LanguageTool** (free, open source).
4. It uses simple **NLP** (Natural Language Processing) to measure the writing:
   it splits the text into paragraphs, sentences and words, then measures vocabulary
   variety, sentence lengths, linking words, keywords and readability.
5. It turns those measurements into marks for five categories and adds them up.
6. It shows the total, a remark, strengths, suggestions, every issue found, and your
   text with mistakes highlighted.
7. You can download the full report as a TXT or Word file.

No paid APIs and no API keys are used.

## 2. Folder structure

```
writing_evaluator/
├── app.py                     # The Streamlit web app (the user interface)
├── requirements.txt           # Python libraries to install
├── README.md                  # This guide
├── .streamlit/
│   └── config.toml            # Colours/theme and 5 MB upload limit
├── utils/
│   ├── __init__.py            # Makes "utils" a Python package
│   ├── file_reader.py         # Reads TXT/DOCX files and handles bad uploads
│   ├── text_analyzer.py       # NLP: paragraphs, sentences, words, vocabulary, readability
│   ├── grammar_checker.py     # LanguageTool grammar/spelling/punctuation checking
│   ├── scorer.py              # The marking scheme: turns analysis into scores
│   └── report_generator.py    # Builds the downloadable TXT and Word reports
└── sample_files/
    ├── sample_essay.txt           # A well-written essay (scores highly)
    ├── sample_with_mistakes.txt   # An essay with many mistakes (scores low)
    └── empty_file.txt             # Used to test the "empty file" message
```

## 3. Requirements

* **Python 3.9 or newer** – <https://www.python.org/downloads/>
* **Java 17 or newer** – needed to run LanguageTool on your computer.
  Free download: <https://adoptium.net/> (choose the latest LTS version).
* Internet connection for the first run (LanguageTool, about 250 MB, downloads automatically once).

`requirements.txt`:

```
streamlit>=1.50
language-tool-python>=2.8
python-docx>=1.1
textstat>=0.7.3
pandas>=2.0
```

| Library | What it is used for |
|---|---|
| streamlit | Builds the web page with simple Python code |
| language-tool-python | Grammar, spelling and punctuation checking |
| python-docx | Reading uploaded Word files and creating the Word report |
| textstat | Readability scores (reading ease, grade level) |
| pandas | Tables and charts in the app |

## 4. Installation, step by step

**Step 1 – Check Python and Java are installed.** Open a terminal (Command Prompt on Windows) and type:

```bash
python --version
java -version
```

Both should print a version number (Java should be 17 or higher). On some Macs/Linux use `python3`.

**Step 2 – Go to the project folder.**

```bash
cd path/to/writing_evaluator
```

**Step 3 – Create a virtual environment** (a private space for this project's libraries).

```bash
python -m venv venv
```

**Step 4 – Activate it.**

* Windows: `venv\Scripts\activate`
* macOS / Linux: `source venv/bin/activate`

You will see `(venv)` at the start of the line.

**Step 5 – Install the libraries.**

```bash
pip install -r requirements.txt
```

**Step 6 – Run the app.**

```bash
streamlit run app.py
```

Your browser opens at <http://localhost:8501>. If not, open that address yourself.

**Step 7 – Try it.** Upload `sample_files/sample_essay.txt`. The first evaluation takes
longer (LanguageTool downloads and starts). After that, results appear in a few seconds.
To stop the app, press `Ctrl + C` in the terminal.

## 5. How the scoring works

| Category | Marks | What is measured |
|---|---|---|
| Grammar & Spelling | 25 | Grammar and spelling mistakes per 100 words (0 mistakes = 25, 8+ per 100 words = 0) |
| Vocabulary | 20 | Variety of words (10), use of longer/richer words (6), no overused words (4) |
| Sentence Structure | 20 | Average sentence length 12–22 words (8), mix of lengths and openings (6), few run-ons/fragments (6) |
| Content & Relevance | 25 | Length/development (8), paragraphs (5), linking words (5), relevance to the topic (7) |
| Punctuation | 10 | Punctuation/capital-letter errors (5), sentences ending properly (2), commas (2), not too many "!" (1) |

**Remarks:** Excellent (90+), Very Good (75–89), Good (60–74), Average (45–59), Needs Improvement (below 45).

**Tip:** type the essay topic or question in the *Topic* box. The app then checks whether the
writing uses the topic's key words. Without a topic, relevance is only estimated.

All the rules live in `utils/scorer.py` as simple, commented `if` checks, so you can change
any mark or threshold easily.

## 6. How errors are handled

| Situation | What the app does |
|---|---|
| No file uploaded | Shows instructions and waits |
| Wrong file type (PDF, image, .doc…) | Explains which formats are allowed |
| Empty file / file with only spaces | "The uploaded file is empty" / "no readable text" message |
| Damaged or fake DOCX | Explains the file could not be opened |
| Fewer than 30 words / more than 5,000 words | Asks for a suitable length |
| File bigger than 5 MB | Blocked by the uploader with a message |
| Java missing or LanguageTool fails | Tries the free online LanguageTool server; if that also fails, uses basic built-in checks and shows a warning |

## 7. Troubleshooting

* **"LanguageTool could not be started" warning** – install Java 17+ and restart the app.
  Check with `java -version`.
* **First run is slow** – normal; LanguageTool is downloading (~250 MB) once.
* **`streamlit` is not recognised** – activate the virtual environment (Step 4) or run
  `python -m streamlit run app.py`.
* **British vs American spelling** – choose the right option in the sidebar so words like
  "colour" or "color" are not marked wrong.

## 8. Limitations (why the score is only an estimate)

* It cannot understand meaning, so a grammatically perfect essay full of wrong facts can still score well.
* Creativity, humour, voice and argument strength are not measured.
* Relevance is based on keywords, not real understanding of the topic.
* Grammar checkers sometimes flag correct sentences or miss real mistakes.

Use the score to spot patterns and practise, not as a final grade.
~~~~

## `sample_files/empty_file.txt`

Create this as an empty file (zero bytes). It is used to test the empty-upload error message.
