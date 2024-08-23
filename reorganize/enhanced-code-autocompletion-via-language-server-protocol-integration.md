Current code autocompletion tools primarily rely on contextual information limited to the active file, recently accessed files, and the cursor's position within the code. This approach often overlooks broader project structures, established coding conventions, and stylistic nuances inherent to the codebase.

One contributing factor to this limitation is the maximum context length constraint inherent to large language models (LLMs). To address these shortcomings, we propose an innovative methodology that harnesses the Language Server Protocol (LSP) to provide a more comprehensive and intelligent autocompletion experience:

1. **Preprocessing Phase**: Utilizing the LSP as a foundation, we perform an extensive scan of the entire project structure. This process constructs an information graph in an incremental, lazy, and parallel manner. Each node within this graph is represented by a concise, text-based summary that encapsulates its corresponding code element.

2. **Autocompletion Phase**: During autocompletion, our engine consults this graph to retrieve information that is most pertinent to the current coding context. This context is not merely textual but is structured at the Abstract Syntax Tree (AST) level, encompassing symbols, definitions, and types.

The crux of our approach lies in the synergy between prompting by AST and querying by AST, enabling a more intelligent and context-aware autocompletion system that aligns with the developer's intent and the project's coding standards.

