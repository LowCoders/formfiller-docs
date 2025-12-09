# FormFiller Documentation

MkDocs Material documentation for the FormFiller form management system.

## Documentation

📖 **Live Documentation:**

- 🇬🇧 **English**: [https://lowcoders.github.io/formfiller-docs/](https://lowcoders.github.io/formfiller-docs/)
- 🇭🇺 **Magyar**: [https://lowcoders.github.io/formfiller-docs/hu/](https://lowcoders.github.io/formfiller-docs/hu/)

## Local Development

### Prerequisites

- Python 3.8+
- pip

### Setup

```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Start development server
mkdocs serve
```

The documentation will be available at `http://localhost:8000`

### Build

```bash
mkdocs build
```

The static site will be generated in the `site/` directory.

## Structure

```
formfiller-docs/
├── docs/
│   ├── en/          # English documentation
│   └── hu/          # Hungarian documentation
├── mkdocs.yml       # MkDocs configuration
├── requirements.txt # Python dependencies
└── README.md        # This file
```

## Source Files

- 🇬🇧 [English documentation](docs/en/index.md)
- 🇭🇺 [Magyar dokumentáció](docs/hu/index.md)

## License

MIT

