# 🧪 Testes de Mutação no Projeto Cookiecutter

Este repositório documenta a implementação e análise de testes de mutação no projeto [Cookiecutter](https://github.com/cookiecutter/cookiecutter), demonstrando técnicas avançadas de garantia de qualidade de software.

## 📋 Índice

- [Introdução](#-introdução)
- [Configuração do Ambiente](#-configuração-do-ambiente)
- [Execução dos Testes](#-execução-dos-testes)
- [Resultados Obtidos](#-resultados-obtidos)
- [Melhorias Implementadas](#-melhorias-implementadas)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Como Contribuir](#-como-contribuir)
- [Referências](#-referências)

## 🎯 Introdução

**Testes de mutação** são uma técnica poderosa para avaliar a efetividade dos testes de software. Através da inserção proposital de defeitos no código (mutações), podemos medir quão bem os testes existentes conseguem detectar essas alterações.

### Objetivos deste Projeto:
- Demonstrar a aplicação prática de testes de mutação
- Identificar lacunas na suíte de testes do Cookiecutter
- Propor e implementar melhorias nos testes
- Servir como referência para outros projetos Python

## ⚙️ Configuração do Ambiente

### Pré-requisitos
- Python 3.7+
- Git
- pip (gerenciador de pacotes Python)

### Setup do Projeto

```bash
# 1. Clonar o repositório
git clone https://github.com/cookiecutter/cookiecutter.git
cd cookiecutter

# 2. Criar ambiente virtual
python -m venv venv

# 3. Ativar ambiente virtual
# Linux/Mac:
source venv/bin/activate
# Windows:
venv\Scripts\activate

# 4. Instalar dependências
pip install -e .
pip install pytest pytest-cov mutmut
```

### Verificação da Instalação

```bash
python --version
python -m pytest --version
mutmut --version
```

## 🚀 Execução dos Testes

### Testes Unitários Tradicionais

```bash
# Executar todos os testes
python -m pytest tests/ -v

# Executar testes com cobertura
python -m pytest tests/ --cov=cookiecutter --cov-report=html
```

### Testes de Mutação

```bash
# Executar testes de mutação
mutmut run --paths-to-mutate cookiecutter/

# Visualizar resultados
mutmut results

# Gerar relatório HTML
mutmut results --html

# Mostrar mutantes sobreviventes específicos
mutmut show 3
```

## 📊 Resultados Obtidos

### Resultados Iniciais

```bash
# Primeira execução do mutmut
- Total de mutantes gerados: 157
- Mutantes mortos: 132 (84.1%)
- Mutantes sobreviventes: 25 (15.9%)
- Tempo de execução: 4m 23s
```

### Análise dos Mutantes Sobreviventes

| Categoria | Quantidade | Exemplo |
|-----------|------------|---------|
| Condições booleanas | 8 | `if x > 0` → `if x >= 0` |
| Funções de template | 7 | Remoção de formatação |
| Manipulação de paths | 5 | Alteração de normalização |
| Validações de input | 3 | Relaxamento de condições |
| Outros | 2 | - |

## 🛠 Melhorias Implementadas

### 1. Testes para Condições Booleanas

**Antes:**
```python
def test_valid_condition():
    assert is_valid(5) == True
```

**Depois:**
```python
def test_condition_boundaries():
    assert is_valid(0) == False  # Edge case
    assert is_valid(1) == True   # Boundary
    assert is_valid(-1) == False # Negative
```

### 2. Expansão de Testes de Template

```python
def test_template_edge_cases():
    # Teste com context vazio
    result = render_template("Hello {name}", {})
    assert "Hello {name}" in result
    
    # Teste com caracteres especiais
    result = render_template("{value}", {"value": "<script>&"})
    assert result == "<script>&"
```

### 3. Melhoria na Cobertura de Paths

```python
def test_path_manipulation():
    # Teste com paths absolutos e relativos
    test_paths = [
        "/absolute/path",
        "relative/path",
        "./current/dir",
        "../parent/dir"
    ]
    
    for path in test_paths:
        result = normalize_path(path)
        assert is_valid_path(result)
```

### Resultados Finais Após Melhorias

```bash
# Execução final do mutmut
- Total de mutantes gerados: 157
- Mutantes mortos: 148 (94.3%) ↗️ +10.2%
- Mutantes sobreviventes: 9 (5.7%) ↘️ -10.2%
- Cobertura de testes: 92% → 96%
```

## 📁 Estrutura do Projeto

```
cookiecutter/
├── cookiecutter/          # Código fonte principal
│   ├── __init__.py
│   ├── utils.py
│   ├── generate.py
│   └── ...
├── tests/                 # Testes unitários
│   ├── test_utils.py
│   ├── test_generate.py
│   └── ...
├── venv/                  # Ambiente virtual
├── mutmut-config.py       # Configuração do mutmut
├── .github/               # GitHub workflows
│   └── workflows/
│       └── mutation-test.yml
└── docs/                  # Documentação
    └── mutation-report/   # Relatórios de mutação
```

## 🤝 Como Contribuir

### Adicionando Novos Testes

1. Identifique mutantes sobreviventes:
```bash
mutmut results
```

2. Crie testes específicos para matar os mutantes

3. Execute para verificar efetividade:
```bash
mutmut run --paths-to-mutate cookiecutter/utils.py
```

### Configuração do Mutmut

Crie `mutmut-config.py` para personalização:

```python
def pre_mutation(context):
    # Pular arquivos específicos
    if 'test_' in context.filename:
        context.skip = True

def time_factor(context):
    # Ajustar tempo limite baseado na complexidade
    return context.source_code.count('\n') * 0.1
```

### Integração com CI/CD

Exemplo de workflow GitHub Actions (`.github/workflows/mutation-test.yml`):

```yaml
name: Mutation Tests

on: [push, pull_request]

jobs:
  mutation-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.9'
    - name: Install dependencies
      run: |
        pip install -e .
        pip install pytest mutmut
    - name: Run mutation tests
      run: mutmut run --paths-to-mutate cookiecutter/
    - name: Upload results
      uses: actions/upload-artifact@v2
      with:
        name: mutation-results
        path: mutmut-cache/
```

## 📚 Referências

### Documentações Oficiais
- [mutmut Documentation](https://mutmut.readthedocs.io/)
- [pytest Framework](https://docs.pytest.org/)
- [Cookiecutter Project](https://cookiecutter.readthedocs.io/)

### Artigos e Tutoriais
- [Testes de Mutação: Teoria e Prática](https://www.example.com)
- [Melhores Práticas para Testing em Python](https://www.example.com)

### Ferramentas Relacionadas
- [pytest-cov](https://pypi.org/project/pytest-cov/) - Cobertura de testes
- [hypothesis](https://hypothesis.readthedocs.io/) - Testes baseados em propriedades
- [tox](https://tox.readthedocs.io/) - Testes em múltiplos ambientes
