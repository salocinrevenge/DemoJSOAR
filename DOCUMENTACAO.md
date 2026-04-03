# DemoJSOAR - Documentação Completa do Projeto

## 📋 Visão Geral

**DemoJSOAR** é um projeto de demonstração que integra a arquitetura cognitiva **JSOAR** (Java Soar) com o ambiente virtual **WS3D** (World Server 3D). O projeto foi desenvolvido como material educativo para o curso **IA941 - Laboratório de Arquiteturas Cognitivas** da Universidade Estadual de Campinas (UNICAMP).

### Objetivo
Demonstrar como implementar um agente inteligente que utiliza a arquitetura cognitiva SOAR em um ambiente virtual 3D interativo, permitindo que o agente perceba, raciocine e aja de forma autônoma.

---

## 📁 Estrutura de Diretórios

### `/src` - Código Fonte Principal

Este é o diretório raiz que contém todo o código-fonte do projeto, organizado em dois subdiretórios principais:

#### `/src/main/java` - Código Java Executável

Contém todos os pacotes e classes Java do projeto:

##### 📦 **Pacote `Simulation`**
Responsável pela simulação e inicialização do ambiente.

- **`Main.java`**: Classe principal de entrada do programa
  - Inicializa os loggers do SOAR e da aplicação
  - Carrega as regras SOAR do arquivo JAR
  - Instancia o ambiente virtual (Environment)
  - Cria a ponte com SOAR (SoarBridge)
  - Inicializa a interface visual (MindView)
  - Loop principal que executa passos de simulação e atualiza a interface

- **`Environment.java`**: Gerenciador do ambiente virtual WS3D
  - Conecta-se ao proxy WS3D (simulador 3D)
  - Cria e gerencia a criatura controlada pelo agente
  - Inicializa o mundo e cria objetos (blocos, itens, etc.)
  - Fornece interface para manipular o ambiente

##### 📦 **Pacote `SoarBridge`**
Implementa a ponte de comunicação entre SOAR e o ambiente WS3D.

- **`SoarBridge.java`**: Classe principal da ponte SOAR
  - Gerencia a instância do agente SOAR
  - Mapeia sensores do ambiente para a "input link" (link de entrada) do SOAR
  - Mapeia comandos da "output link" (link de saída) do SOAR para ações no ambiente
  - Executa ciclos de raciocínio do SOAR
  - Mantém sincronização entre o estado do ambiente e a memória de trabalho do SOAR

- **`Command.java`**: Classe abstrata base para comandos
  - Define a interface padrão para todos os comandos que podem ser executados
  - Implementa métodos abstratos para execução e validação de comandos

- **`CommandMove.java`**: Comando de movimento
  - Implementa movimento da criatura em direções específicas
  - Interage com o WS3D para mover o agente no ambiente

- **`CommandEat.java`**: Comando de consumo
  - Permite que a criatura consuma alimentos ou recursos
  - Implementa a lógica de interação com objetos comestíveis

- **`CommandGet.java`**: Comando de coleta
  - Permite que a criatura pegue/colete objetos do ambiente
  - Gerencia a interação com itens coletáveis

##### 📦 **Pacote `support`**
Funções de suporte, visualização e utilitários.

- **`MindView.java`** (e `MindView.form`): Interface gráfica principal
  - Janela Swing que exibe a visualização do ambiente
  - Mostra em tempo real o estado da criatura
  - Visualiza a árvore de estrutura de memória do SOAR
  - Controles para pausar/retomar a simulação
  - Conexão com o painel de visualização do WS3D

- **`WorkingMemoryViewer.java`** (e `WorkingMemoryViewer.form`): Visualizador de memória
  - Exibe a estrutura completa da "working memory" (memória de trabalho) do SOAR
  - Mostra entrada (input link) e saída (output link) em formato de árvore
  - Permite inspeção em tempo real do estado do agente

- **`RendererJTree.java`**: Renderizador customizado para árvores
  - Renderiza elementos da memória de trabalho em uma JTree (árvore Swing)
  - Exibe a hierarquia de estruturas de forma visual

- **`TreeElement.java`**: Elemento de árvore
  - Classe que representa um nó na visualização da árvore de memória

- **`NativeUtils.java`**: Utilitários nativos
  - Carrega recursos do arquivo JAR (como regras SOAR)
  - Fornece funcionalidades para acessar arquivos empacotados na aplicação

#### `/src/main/resources` - Recursos da Aplicação

Contém arquivos não-Java necessários para execução:

##### `images/` - Imagens e Ícones
- `pause-icon.png`: Ícone para botão de pausa
- `play-icon.png`: Ícone para botão de play
- Outras imagens utilizadas na interface gráfica

##### `rules/` - Regras SOAR
Contém os arquivos de regras que definem o comportamento do agente:

- **`soar-rules.soar`**: Conjunto padrão de regras de produção SOAR
  - Define como o agente raciocina
  - Mapeia entradas do ambiente a ações
  - Implementa estratégias de resolução de problemas

- **`soar-rules-ativ1.soar`**: Regras específicas para atividade 1
  - Variações ou extensões para demonstrações específicas
  - Pode incluir tarefas ou comportamentos customizados

---

### `/build` - Diretório de Build

Contém artefatos gerados durante a compilação (NÃO deve ser editado manualmente):

- **`classes/java/main`**: Classes Java compiladas
  - Bytecode compilado dos pacotes Simulation, SoarBridge e support

- **`distributions`**: Distribuições empacotadas
  - Arquivos preparados para distribuição

- **`libs`**: Bibliotecas compiladas em JAR

- **`reports`**: Relatórios de build
  - `problems-report.html`: Relatório de problemas e avisos de compilação

- **`resources`**: Recursos processados
  - Cópia dos recursos do `src/main/resources` após processamento

- **`scripts`**: Scripts executáveis gerados
  - `DemoJSOAR`: Script para Linux/Mac
  - `DemoJSOAR.bat`: Script para Windows

---

### `/gradle` - Configuração do Gradle

Contém configuração do sistema de build Gradle:

- **`wrapper/gradle-wrapper.properties`**: Propriedades do Gradle Wrapper
  - Define qual versão do Gradle usar
  - Permite que o projeto funcione sem Gradle instalado

---

## 📄 Arquivos de Configuração

### `build.gradle` - Configuração do Build
Define como o projeto é compilado e empacotado:

```gradle
Plugins utilizados:
- java: Compilação Java
- jacoco: Cobertura de testes
- application: Criação de aplicação executável

Propriedades principais:
- mainClass: Simulation.Main (classe principal)
- Repositórios: Maven Central e JitPack

Dependências principais:
- jsoar-core (4.1.3): Engine SOAR
- jsoar-debugger: Depurador SOAR
- WS3DProxy (0.0.7): Proxy para comunicação com WS3D
- Swing/GUI libraries: Interface visual
- logback: Logging
- jackson: Processamento JSON

Configuração JAR:
- Empacota todas as dependências
- Define manifesto com classe principal
- Exclui arquivos de assinatura digital
```

### `gradle.properties` - Propriedades do Projeto
Define propriedades globais, como versões e configurações.

### `settings.gradle` - Configurações do Gradle
Define nome do projeto e configurações do workspace Gradle.

### `build.gradle` e `gradlew` / `gradlew.bat`
- Build script e wrappers do Gradle para garantir compatibilidade

---

## 🔧 Dependências Principais

| Dependência | Versão | Função |
|-----------|--------|--------|
| **jsoar-core** | 4.1.3 | Engine da arquitetura cognitiva SOAR |
| **jsoar-debugger** | 4.1.3 | Interface de depuração para SOAR |
| **WS3DProxy** | 0.0.7 | Proxy para comunicação com ambiente WS3D |
| **logback-classic** | 1.3.6 | Framework de logging |
| **docking-frames-common** | 1.1.2-P19c | Framework para layouts flutuantes em Swing |
| **swingx-all** | 1.6.5-1 | Extensões para Swing |
| **jackson-databind** | 2.15.1 | Processamento JSON |
| **re2j** | 1.7 | Regex engine melhorado |
| **junit** | 4.13 | Framework de testes unitários |

---

## 🔄 Fluxo de Execução

```
1. Main.java inicia a aplicação
   ↓
2. Environment cria conexão com WS3D e inicializa o mundo
   ↓
3. SoarBridge cria agente SOAR e carrega regras
   ↓
4. MindView cria interface gráfica e conecta aos componentes
   ↓
5. Loop principal:
   a. SoarBridge lê sensores do ambiente
   b. Mapeia sensores para input link do SOAR
   c. SOAR executa um ciclo de raciocínio
   d. Mapeia output link para comandos no ambiente
   e. Ambiente executa comandos e atualiza estado
   f. GUI atualiza visualização
   g. Volta ao passo 5.a
```

---

## 🎯 Componentes Principais

### SOAR (Symbolic Architectures for Advanced Intelligence)
- Arquitetura cognitiva implementada em Java via JSOAR
- Usa memória de trabalho e produção de regras
- Executa ciclos de decisão baseados em regras

### WS3D (World Server 3D)
- Ambiente virtual 3D para simulação
- Gerencia criatura, objetos e físicas do mundo
- Comunica via proxy (WS3DProxy)

### Ponte de Integração (SoarBridge)
- Sincroniza sensores WS3D com input link SOAR
- Mapeia decisões SOAR para comandos WS3D
- Mantém consistência entre ambos os sistemas

### Interface Visual (MindView)
- Visualiza o ambiente 3D em tempo real
- Mostra estrutura da memória de trabalho SOAR
- Permite controle manual de simulação (play/pause)

---

## 📝 Regras SOAR

As regras SOAR (arquivos `.soar`) definem o comportamento do agente:

- **Sensor mappings**: Como interpretar entrada do ambiente
- **Decision rules**: Como o agente toma decisões
- **Action mappings**: Como converter decisões em ações
- **Memory management**: Limpeza e organização da memória

---

## 🚀 Como Usar

### Compilar o Projeto
```bash
./gradlew build
```

### Executar
```bash
./gradlew run
# ou
./build/scripts/DemoJSOAR  # Linux/Mac
./build/scripts/DemoJSOAR.bat  # Windows
```

### Depurar
- Use o jSOAR Debugger integrado
- Inspecione working memory via WorkingMemoryViewer
- Monitore entrada/saída em MindView

---

## 📚 Referências

- **JSOAR**: https://github.com/soartech/jsoar
- **WS3D**: https://github.com/CST-Group/ws3d
- **Curso IA941**: https://faculty.dca.fee.unicamp.br/gudwin/courses/IA941
- **SOAR Official**: https://soar.eecs.umich.edu/

---

## 📝 Autores

- Danilo Lucentini
- Ricardo Gudwin
- Universidade Estadual de Campinas (UNICAMP)

---

## 📄 Licença

Consulte o arquivo [LICENSE](LICENSE) para detalhes.
