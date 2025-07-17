# AI Essay Writer 📝🤖

An intelligent multi-agent essay writing system that combines AI-powered planning, research, writing, and revision to create high-quality essays automatically.

## 🌟 Features

- **Multi-Agent Architecture** - Specialized agents for planning, research, writing, and critique
- **Automated Research** - Real-time web research using Tavily API
- **Iterative Improvement** - Automatic revision cycles with AI-powered feedback
- **Interactive GUI** - User-friendly interface for essay generation
- **Persistent Memory** - SQLite-based state management for workflow continuity
- **Structured Workflow** - Graph-based execution with conditional logic

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- OpenAI API key
- Tavily API key (for web research)

### Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/ai-essay-writer.git
cd ai-essay-writer
```

2. Install dependencies:
```bash
pip install langgraph langchain-openai tavily-python python-dotenv gradio
```

3. Set up environment variables:
```bash
# Create .env file
echo "OPENAI_API_KEY=your_openai_api_key_here" >> .env
echo "TAVILY_API_KEY=your_tavily_api_key_here" >> .env
```

### Usage

#### Jupyter Notebook
```bash
jupyter notebook essay_writer.ipynb
```

#### Python Script
```python
from essay_writer import EssayWriter

# Initialize the essay writer
writer = EssayWriter()

# Generate an essay
result = writer.generate_essay(
    task="What is the difference between LangChain and LangSmith?",
    max_revisions=2
)

print(result['draft'])
```

## 🏗️ Architecture

### Multi-Agent Workflow

```
Entry Point: Planner
    ↓
Research Plan → Generate Essay
    ↓              ↓
    └─────→ Should Continue? ←─────┘
              ↓ (if revisions < max)
           Reflect
              ↓
         Research Critique
              ↓
         Generate (Revised)
```

### Agent Roles

#### 1. **Planner Agent**
- **Role**: Creates high-level essay outline
- **Input**: Essay topic/task
- **Output**: Structured outline with sections and notes
- **Prompt**: Expert writer creating essay outlines

#### 2. **Research Plan Agent**
- **Role**: Generates research queries for initial planning
- **Input**: Essay task
- **Output**: List of search queries (max 3)
- **Integration**: Tavily web search API

#### 3. **Generation Agent**
- **Role**: Writes and revises essays
- **Input**: Task, plan, research content, critique (if revision)
- **Output**: Complete 5-paragraph essay
- **Model**: GPT-3.5-turbo with temperature=0

#### 4. **Reflection Agent**
- **Role**: Provides critique and improvement suggestions
- **Input**: Essay draft
- **Output**: Detailed feedback on length, depth, style
- **Persona**: Teacher grading essays

#### 5. **Research Critique Agent**
- **Role**: Researches information for revisions
- **Input**: Critique feedback
- **Output**: Additional research content
- **Integration**: Tavily web search API

## 📊 State Management

### AgentState Schema
```python
class AgentState(TypedDict):
    task: str              # Original essay prompt
    plan: str              # Essay outline
    draft: str             # Current essay draft
    critique: str          # Feedback from reflection
    content: List[str]     # Research content
    revision_number: int   # Current revision count
    max_revisions: int     # Maximum allowed revisions
```

### Memory System
- **Backend**: SQLite with LangGraph checkpointer
- **Persistence**: Workflow state maintained across sessions
- **Threading**: Support for multiple concurrent essays

## 🔧 Configuration

### Model Settings
- **Primary Model**: GPT-3.5-turbo
- **Temperature**: 0 (deterministic output)
- **Structured Output**: Pydantic models for query generation

### Research Settings
- **Search Provider**: Tavily API
- **Results per Query**: 2 maximum
- **Query Limit**: 3 queries per research phase

### Revision Control
- **Default Max Revisions**: 2
- **Termination**: Automatic after max revisions reached
- **Conditional Logic**: Continue/end based on revision count

## 🛠️ Technical Components

### Core Dependencies
```python
langgraph              # Multi-agent workflow orchestration
langchain-openai       # OpenAI model integration
tavily-python          # Web research API
python-dotenv          # Environment variable management
gradio                 # GUI interface (optional)
```

### Prompt Templates

#### Planning Prompt
```python
PLAN_PROMPT = """You are an expert writer tasked with writing a high level outline of an essay. 
Write such an outline for the user provided topic. Give an outline of the essay along with any relevant notes 
or instructions for the sections."""
```

#### Writing Prompt
```python
WRITER_PROMPT = """You are an essay assistant tasked with writing excellent 5-paragraph essays.
Generate the best essay possible for the user's request and the initial outline. 
If the user provides critique, respond with a revised version of your previous attempts."""
```

#### Reflection Prompt
```python
REFLECTION_PROMPT = """You are a teacher grading an essay submission. 
Generate critique and recommendations for the user's submission. 
Provide detailed recommendations, including requests for length, depth, style, etc."""
```

## 🌐 Web Research Integration

### Tavily API Features
- Real-time web search
- Content extraction and summarization
- Relevant result filtering
- Rate-limited queries for efficiency

### Research Workflow
1. **Initial Research**: Based on essay topic
2. **Critique Research**: Based on revision feedback
3. **Content Integration**: Seamlessly woven into essay generation

## 🎨 User Interface

### GUI Features (via helper module)
- Interactive essay topic input
- Real-time generation progress
- Revision history viewing
- Export capabilities
- Responsive design

### Interface Components
```python
from helper import ewriter, writer_gui

# Initialize multi-agent system
MultiAgent = ewriter()

# Launch GUI
app = writer_gui(MultiAgent.graph)
app.launch()
```

## 📝 Essay Structure

### Standard Format
- **5-paragraph structure**
- Introduction with thesis
- Three supporting body paragraphs
- Conclusion with summary

### Quality Metrics
- Coherent argument flow
- Evidence-based claims
- Proper citation integration
- Academic writing style

## 🔄 Revision Process

### Automatic Improvement Cycle
1. **Initial Draft**: Based on plan and research
2. **AI Critique**: Detailed feedback generation
3. **Additional Research**: Target specific improvements
4. **Revised Draft**: Incorporates feedback and new research
5. **Repeat**: Until max revisions reached

### Critique Categories
- Content depth and accuracy
- Argument structure and logic
- Writing style and clarity
- Length and formatting
- Evidence and support

## 🚧 Development

### Extending the System

#### Adding New Agents
```python
def custom_agent_node(state: AgentState):
    # Custom agent logic
    messages = [
        SystemMessage(content="Custom prompt"),
        HumanMessage(content=state['task'])
    ]
    response = model.invoke(messages)
    return {"custom_field": response.content}

# Add to graph
builder.add_node("custom_agent", custom_agent_node)
```

#### Custom Prompts
Modify the prompt constants to change agent behavior:
```python
CUSTOM_PROMPT = """Your custom instructions here..."""
```

#### Alternative Models
```python
from langchain_anthropic import ChatAnthropic
model = ChatAnthropic(model="claude-3-sonnet-20240229")
```

## 📋 API Requirements

### OpenAI API
- **Models**: GPT-3.5-turbo, GPT-4 (configurable)
- **Features**: Chat completions, structured output
- **Pricing**: Per-token usage

### Tavily API
- **Service**: Web search and content extraction
- **Limits**: Rate limiting applies
- **Features**: Real-time search, content summarization

## 🔐 Security & Privacy

- API keys stored in environment variables
- No persistent storage of user content
- Memory cleared between sessions
- Secure API communication

## 🐛 Troubleshooting

### Common Issues

1. **API Key Errors**
   ```bash
   # Verify environment variables
   python -c "import os; print(os.getenv('OPENAI_API_KEY'))"
   ```

2. **Import Errors**
   ```bash
   # Install missing dependencies
   pip install --upgrade langgraph langchain-openai
   ```

3. **Research Failures**
   ```bash
   # Check Tavily API status
   python -c "from tavily import TavilyClient; print('Tavily OK')"
   ```

### Debug Mode
Enable detailed logging:
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## 📊 Performance Metrics

### Typical Processing Times
- **Planning**: 5-10 seconds
- **Research**: 10-15 seconds per phase
- **Generation**: 15-30 seconds
- **Reflection**: 5-10 seconds
- **Total**: 2-5 minutes for complete essay

### Resource Usage
- **API Calls**: 5-15 per essay
- **Memory**: ~50MB during execution
- **Storage**: Minimal (SQLite state only)

## 🔮 Roadmap

- [ ] Support for different essay types (argumentative, narrative, etc.)
- [ ] Citation formatting (MLA, APA, Chicago)
- [ ] Plagiarism detection integration
- [ ] Multi-language support
- [ ] Advanced research source filtering
- [ ] Export to various formats (PDF, Word, etc.)
- [ ] Collaborative editing features
- [ ] Template customization
- [ ] Academic integrity tools

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Add your improvements
4. Test with various essay topics
5. Submit a pull request

## 🆘 Support

If you encounter issues:

1. Check API key configuration
2. Verify internet connectivity
3. Review dependency versions
4. Check API service status
5. Open an issue with error details

---

**Write smarter, not harder!** ✍️✨

*Built with ❤️ using LangGraph and AI agents*
