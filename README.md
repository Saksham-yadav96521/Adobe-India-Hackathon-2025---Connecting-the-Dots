# Adobe Hackathon Task 1A: PDF Outline Extraction

A high-performance, CPU-optimized solution for extracting structured outlines from PDF documents.

## Overview

This solution extracts document titles and hierarchical headings (H1, H2, H3) from PDF files using a multi-method approach that balances speed and accuracy.

## Features

- **Multi-Method Extraction**: Uses PyMuPDF for existing bookmarks, falls back to font-based analysis
- **Fast Performance**: Optimized for ≤10 second processing of 50-page PDFs
- **CPU Optimized**: Runs efficiently on AMD64 architecture without GPU requirements
- **Robust Analysis**: Intelligent font size and formatting analysis for heading detection
- **Docker Ready**: Containerized solution with all dependencies included

## Architecture

### Method 1: PyMuPDF Outline Extraction (Primary)
- Extracts existing PDF bookmarks/table of contents
- Fastest method when bookmarks are available
- Near-instantaneous processing

### Method 2: Font-Based Analysis (Fallback)
- Analyzes font sizes, styles, and positioning across the document
- Uses statistical analysis to identify heading fonts vs body text
- Groups characters by lines and applies heading detection heuristics

### Method 3: Title Extraction
- Extracts titles from PDF metadata
- Falls back to first-page analysis for largest/centered text
- Smart cleaning and validation of extracted titles

## Algorithm Details

### Font Analysis Heuristics
1. **Statistical Font Analysis**: Identifies most common font size as body text
2. **Heading Font Detection**: Finds fonts significantly larger than body text
3. **Hierarchical Classification**: Maps font sizes to H1, H2, H3 levels
4. **Position-Based Filtering**: Considers text positioning and formatting

### Heading Level Determination
- **H1**: Largest font size, typically centered or prominent
- **H2**: Second largest font size, often bold
- **H3**: Third largest font size, smaller than H2 but larger than body

### Text Cleaning
- Removes page numbers, chapter prefixes, and formatting artifacts
- Validates heading length (3-200 characters)
- Normalizes whitespace and line breaks

## Dependencies

- **PyMuPDF (1.23.27)**: Fast PDF processing and outline extraction
- **pdfplumber (0.11.2)**: Detailed character-level PDF analysis
- **pdfminer.six (20231228)**: Low-level PDF parsing capabilities

## Docker Usage

### Build the Image
```bash
docker build --platform linux/amd64 -t pdf-outline-extractor:latest .
```

### Run the Container
```bash
docker run --rm \
  -v $(pwd)/input:/app/input \
  -v $(pwd)/output:/app/output \
  --network none \
  pdf-outline-extractor:latest
```
See [`BUILD_INSTRUCTIONS.md`](BUILD_INSTRUCTIONS.md) for a full quick start guide.

## Input/Output Format

### Input
- Directory: `/app/input`
- Format: PDF files (*.pdf)
- Limit: Up to 50 pages per PDF

### Output
- Directory: `/app/output`
- Format: JSON files with same name as input PDF
- Structure:
```json
{
  "title": "Document Title",
  "outline": [
    {
      "level": "H1",
      "text": "Introduction", 
      "page": 1
    },
    {
      "level": "H2",
      "text": "Background",
      "page": 2
    }
  ]
}
```

## Performance Specifications

- **Processing Time**: ≤10 seconds for 50-page PDF
- **Model Size**: Lightweight libraries, no ML models required
- **Memory Usage**: Optimized for 16GB RAM systems
- **CPU**: Utilizes multi-core processing on 8-CPU systems
- **Architecture**: AMD64 (x86_64) compatible

## Error Handling

- Graceful fallback between extraction methods
- Robust error logging and recovery
- Empty results for failed extractions rather than crashes
- Comprehensive input validation

## Optimization Features

- **Lazy Loading**: Processes pages only when needed
- **Memory Efficient**: Closes resources promptly
- **Parallel Ready**: Can be extended for multi-file parallel processing
- **Caching**: Reuses font analysis across pages

## Testing

The solution has been designed to handle various PDF types:
- Academic papers with clear heading hierarchies
- Business documents with embedded bookmarks
- Scanned documents (where text is selectable)
- Multi-language documents
- Complex layouts with mixed formatting

## Limitations

- Requires selectable text (not pure image-based PDFs)
- Heading detection accuracy depends on consistent font usage
- Limited to H1, H2, H3 levels as per requirements
- No network access for enhanced processing

## Future Enhancements

- Multi-language OCR support
- Advanced layout analysis with computer vision
- Machine learning-based heading classification
- Support for tables and figures in outline


# Challenge 1(b) - Persona-Based Document Analysis and Summarization

## Overview

This challenge implements an intelligent document analysis system that extracts, ranks, and summarizes relevant content from multiple PDF documents based on a specific persona and job requirement. The system is designed to help users quickly find and understand the most relevant information from a collection of documents.

## Problem Statement

Given a collection of PDF documents and a specific persona with a job to be done, the system should:
1. Extract text sections from all PDF documents
2. Rank sections based on relevance to the persona and job
3. Summarize the most important sections
4. Generate a structured output with metadata and analysis

## Example Use Case

**Persona**: Travel Planner  
**Job**: Plan a trip of 4 days for a group of 10 college friends

The system processes travel-related PDF documents about South of France and identifies the most relevant sections for trip planning, then provides summaries to help the travel planner make informed decisions.

## Architecture

The solution consists of three main components:

### 1. Parser (`src/parser.py`)
- Extracts text sections from PDF documents using PyMuPDF
- Identifies document structure and sections
- Analyzes font statistics to determine section hierarchy

### 2. Ranker (`src/ranker.py`)
- Ranks extracted sections based on relevance to the persona and job
- Uses semantic analysis to determine importance
- Returns top-ranked sections for further processing

### 3. Summarizer (`src/summarizer.py`)
- Generates concise summaries of the most relevant sections
- Provides structured analysis of each section
- Maintains document and page references

## Input Data

The system processes PDF documents located in the `input/` directory:
- South of France - Cities.pdf
- South of France - Cuisine.pdf
- South of France - History.pdf
- South of France - Restaurants and Hotels.pdf
- South of France - Things to Do.pdf
- South of France - Tips and Tricks.pdf
- South of France - Traditions and Culture.pdf

## Configuration

The persona and job are configured in `persona_task.json`:
```json
{
  "persona": "Travel Planner",
  "job_to_be_done": "Plan a trip of 4 days for a group of 10 college friends."
}
```

## Output Structure

The system generates a comprehensive JSON output in the `output/` directory containing:

```json
{
  "metadata": {
    "documents": ["list of processed PDF files"],
    "persona": "Travel Planner",
    "job_to_be_done": "Plan a trip of 4 days for a group of 10 college friends",
    "timestamp": "ISO timestamp"
  },
  "extracted_sections": [
    {
      "document": "filename.pdf",
      "page_number": 1,
      "section_title": "Section Title",
      "importance_rank": 1
    }
  ],
  "subsection_analysis": [
    {
      "document": "filename.pdf",
      "page_number": 1,
      "summary": "Generated summary of the section"
    }
  ]
}
```

## Installation

1. Install required dependencies:
```bash
pip install -r requirements.txt
```

2. Ensure your PDF documents are placed in the `input/` directory

3. Configure your persona and job in `persona_task.json`

## Usage

Run the main script:
```bash
python run.py
```

The system will:
1. Process all PDF files in the `input/` directory
2. Extract and analyze text sections
3. Rank sections based on relevance to your persona and job
4. Generate summaries of the most important sections
5. Save the results to `output/output.json`

## Docker Support

Build and run using Docker:
```bash
docker build -t challenge1b .
docker run -v $(pwd)/input:/app/input -v $(pwd)/output:/app/output challenge1b
```

## Key Features

- **Multi-document Processing**: Handles multiple PDF files simultaneously
- **Persona-aware Analysis**: Tailors content extraction based on user persona
- **Intelligent Ranking**: Prioritizes sections most relevant to the job requirements
- **Structured Output**: Provides organized, machine-readable results
- **Metadata Tracking**: Maintains document source and page references
- **Dockerized Deployment**: Easy containerized execution

## Technical Requirements

- Python 3.7+
- PyMuPDF (fitz) for PDF processing
- Additional dependencies listed in `requirements.txt`

## Files Structure

```
challenge1b/
├── src/
│   ├── parser.py          # PDF text extraction
│   ├── ranker.py          # Section ranking logic
│   └── summarizer.py      # Content summarization
├── input/                 # PDF documents to process
├── output/                # Generated analysis results
├── persona_task.json      # Persona and job configuration
├── run.py                 # Main execution script
├── requirements.txt       # Python dependencies
├── Dockerfile            # Container configuration
├── approach_explanation.md # Detailed technical approach
└── README.md             # This file
```

## Contributing

1. Ensure all PDF files are placed in the `input/` directory
2. Update `persona_task.json` with your specific persona and job requirements
3. Run the system and review the generated output
4. Modify the ranking and summarization logic as needed for your use case

## License

This project is part of the Adobe India Hackathon 2025 - Connecting the Dots challenge.
