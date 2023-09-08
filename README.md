# pseudocode_summarizer

## Overview

`pseudocode_summarizer` is a Python library and CLI tool that generates a concise, natural language pseudocode summary of your entire project folder. This summary can serve as an intermediate step in development, documentation writing, or even for embedding your repo for search-and-retrieval or Q&A. The tool leverages GPT-3.5 Turbo to summarize a decent-sized project for just 2-4 cents. Future plans include a Github Action for seamless integration into automated cloud deployment workflows.

## Installation

If you install `pseudocode_summarizer` via `pip`, the CLI tool should be automatically added to your system PATH. If you encounter issues or have installed it manually, you may need to add it to your PATH manually.

### PyPi Installation

```bash
pip install pseudocode-summarizer
```

### Clone from GitHub

```bash
git clone https://github.com/Promptly-Technologies-LLC/pseudocode_summarizer.git
cd pseudocode_summarizer
pip install .
```

## Usage

### Setting an OpenAI API Key

Setting environment variables for local command-line usage can be done in several ways, depending on your operating system and shell. Here are some methods:

#### 1. Exporting in Shell

You can set an environment variable in your shell session. This will set the variable for the duration of the shell session, making it available to `pseudocode_summarizer` in the same session. The exact command differs depending on which shell you use.

In Bash, use the `export` command:

```bash
export OPENAI_API_KEY=your_api_key_here
```

In Windows Command Prompt, you can use the `set` command:

```cmd
set OPENAI_API_KEY=your_api_key_here
```

In Windows PowerShell, you can use the `$env:` prefix:

```powershell
$env:OPENAI_API_KEY="your_api_key_here"
```

#### Using an `.env` File

You can also place your environment variables in a `.env` file in the same directory as the repo to be summarized, with the following text:

```
OPENAI_API_KEY=your_api_key_here
```

Make sure to replace `your_api_key_here` with your actual API key. 

When you run your script, `load_dotenv()` will load these variables into the environment.

#### Making Environment Variables Permanent

If you find yourself needing to set this environment variable frequently, you may want to add the export command to your shell profile script (e.g., `.bashrc`, `.zshrc` for Unix-like systems) or set it as a system-wide environment variable through system settings.

#### Using an option flag

You can also pass your API key as an option flag to the CLI tool (`--api_key="your_openai_api_key"`) or Python API (`api_key="your_openai_api_key"`). For security reasons, this is not recommended, as it will expose your API key in your shell history or command-line history.

### Command-Line Interface

To summarize a project folder, use the `summarize` command in your shell from the folder you want to summarize. The CLI tool will automatically map the project folder, identify the salient files to summarize, and generate a pseudocode summary of the entire project. By default, the tool will create a `docs` folder if one does not already exist, and then will create `project_map.json` and `pseudocode.md` files in that folder. The default paths for these files can be adjusted using the `--pseudocode_file` and `--project_map_file` options.

The `project_map.json` file contains a mapping of the project folder, and the `pseudocode.md` file contains the pseudocode summary of the project as generated by GPT-3.5 Turbo. By default, `pseudocode_summarizer` uses the model with 8k context length; however, it will fall back to 16k if necessary. The tool currently only supports OpenAI's chat models.

Note that the syntax for setting option flags differs slightly depending on your shell. Bash uses an equal sign, while Windows Powershell uses a space, as in the examples below.

#### Bash/Zsh

```bash
summarize --startpath="./my_project" --api_key="your_openai_api_key"
```

#### PowerShell

```powershell
summarize --startpath ".\my_project" --api_key "your_openai_api_key"
```

#### Option Flags

- `--startpath`: Path to the project folder (default is current directory).
- `--pseudocode_file`: Path to the pseudocode summary file (optional).
- `--project_map_file`: Path to the project map file (optional).
- `--include`: Roles to include in the summary (optional).
- `--api_key`: OpenAI API key (required).
- `--model_name`: Name of the chatbot model (optional).
- `--long_context_fallback`: Fallback chatbot for long context (optional).
- `--temperature`: Temperature parameter for the chatbot model (optional).

### Python API

To use the tool from a Python script, import and invoke as follows:

```python
from pseudocode_summarizer import summarize_project_folder

summarize_project_folder(
    startpath="./my_project",
    api_key="your_openai_api_key"
)
```

The `summarize_project_folder` function takes the same arguments as the CLI tool.

The Python API also exposes the `read_pseudocode_file` function and the `ModulePseudocode` class, which can be used to read and parse the pseudocode summary file. The `ModulePseudocode` class is a pydantic model that can be used to validate the JSON file and access its contents. It has the attributes `path`, `modified`, and `content`, which correspond to the path of the module, the last modified timestamp of the module, and the pseudocode summary of the module, respectively. The `read_pseudocode_file` function returns a list of `ModulePseudocode` objects, one for each module in the project folder.

```python
from pseudocode_summarizer import read_pseudocode_file, ModulePseudocode

pseudocode: list[ModulePseudocode] = read_pseudocode_file("./docs/pseudocode.md")
```

## Contributing

We welcome contributions! Feel free to submit pull requests for new features, improvements, or bug fixes. Please make sure to follow best practices and include unit tests using the pytest framework.

For any issues, please [create an issue on GitHub](https://github.com/Promptly-Technologies-LLC/pseudocode_summarizer/issues).

## Modules and Key Functions

- `summarize.py`: Orchestrates the summarization of the entire project folder.
- `cli.py`: Defines the command-line interface for the tool.
- `mapper.py`: Maps the project folder.
- `chatbot.py`: Initializes and queries the chatbot model.
- `classifier.py`: Classifies files' roles by querying the chatbot.
- `summarizer.py`: Summarizes individual files by querying the chatbot.
- `file_handler.py`: Handles reading and writing of pseudocode files.
  
For a more detailed understanding, please refer to the source code, inline comments, and, most importantly, [docs\pseudocode.md](docs\pseudocode.md)!

## To-do

- [ ] Create a Github Action to run the tool on every push
- [ ] Add project-level tech stack summarization
- [ ] Add module-level generation of usage instructions
- [ ] Add support for LLMs other than OpenAI's
- [ ] Make decisions about data structures, as described below

## Ideas

Instead of having separate `ProjectFile` and `FileClassification` data types, have a single class with `modified` and `role` both as optional fields. And instead of a `FileClassificationList` class, I could simplify in the pydantic parser as `[FileClassification]` and convert the `to_json` method to a function that takes a list of `FileClassification` objects. This would require removing the `files` attribute in the list comprehension in `classify_files`. However, this might still cause `modified` to show up in our JSON serialization, filling our LLM context with useless tokens.

Do I want to experiment with YAML instead of JSON to keep context length down?