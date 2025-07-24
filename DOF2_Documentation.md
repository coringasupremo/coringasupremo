# Documentação do DOF2 (Double-O-Files 2) para SA-MP

O **DOF2** (Double-O-Files 2) é uma biblioteca para manipulação de arquivos INI no SA-MP, projetada para alta performance e flexibilidade. Suporta seções, diversos tipos de dados (strings, inteiros, floats, booleanos, hexadecimais e binários) e callbacks para processamento dinâmico. É compatível com formatos de DUDB, DINI e parcialmente y_ini, sendo ideal para sistemas de contas de jogadores e configurações de servidor.

**Data**: 23 de Julho de 2025  
**Autor**: Desenvolvido por Collw, Documentado por Grok 3

## Índice
- [Introdução](#introdução)
- [Características Principais](#características-principais)
- [Configurações Iniciais](#configurações-iniciais)
- [Funções de Manipulação de Arquivos](#funções-de-manipulação-de-arquivos)
- [Funções de Manipulação de Entradas](#funções-de-manipulação-de-entradas)
- [Funções de Manipulação de Seções](#funções-de-manipulação-de-seções)
- [Parsing e Callbacks](#parsing-e-callbacks)
- [Funções Auxiliares](#funções-auxiliares)
- [Macros Úteis](#macros-úteis)
- [Compatibilidade com DUDB e DINI](#compatibilidade-com-dudb-e-dini)
- [Exemplo Completo de Sistema de Contas](#exemplo-completo-de-sistema-de-contas)
- [Dicas e Boas Práticas](#dicas-e-boas-práticas)
- [Limitações](#limitações)
- [Como Hospedar no GitHub](#como-hospedar-no-github)

## Introdução
O DOF2 é uma biblioteca para SA-MP que manipula arquivos INI com alta eficiência. Usa busca binária (O(log n)) para acesso rápido a dados, suporta seções (`[tag]`) e é compatível com DUDB, DINI e parcialmente y_ini. É amplamente usado para gerenciar dados persistentes, como contas de jogadores, posições, scores e configurações de servidor.

### Por que usar o DOF2?
- **Performance**: Busca binária para acesso rápido a entradas.
- **Flexibilidade**: Suporta seções, UTF-8 e vários tipos de dados.
- **Compatibilidade**: Emula DUDB e DINI.
- **Callbacks**: Suporta parsing dinâmico com `OnParseFile`.
- **Ordenação**: Permite ordenar seções e entradas alfabeticamente.

## Características Principais
- **Seções**: Organiza dados em seções `[tag]` (ex.: `[Dados]`, `[Stats]`).
- **Tipos de Dados**: Strings, inteiros, floats, booleanos, hexadecimais e binários.
- **UTF-8**: Suporte a caracteres especiais (ex.: russo, chinês) com `PACK_CONTENT = false`.
- **Callbacks**: Funções `OnParseFile` para processamento dinâmico.
- **Ordenação**: Ordenação alfabética de seções e entradas.
- **Compatibilidade**: Suporta formatos de DUDB, DINI e y_ini.

## Configurações Iniciais
Antes de incluir o DOF2, configure as seguintes constantes no seu código Pawn:

```pawn
#define MAX_SECTION_TAG     (32)    // Tamanho máximo do nome de uma seção.
#define MAX_LINE_SIZE       (150)   // Tamanho máximo de uma linha (chave + valor).
#define MAX_SECTIONS        (32)    // Número máximo de seções (máximo 255).
#define MAX_ENTRIES         (256)   // Número máximo de entradas no cache.
#define MAX_FILE_SIZE       (64)    // Tamanho máximo do nome do arquivo.
#define PACK_CONTENT        (false) // Usa strings compactadas (sem caracteres especiais).
#define USER_FILE_PATH      "Users/%s.ini" // Caminho padrão para arquivos de usuário.
#define USER_PW_HASH_KEY    "password"     // Chave para senhas hasheadas.
```

### Explicação das Configurações
- `MAX_SECTION_TAG`: Comprimento máximo do nome de uma seção (ex.: `[Dados]`).
- `MAX_LINE_SIZE`: Limite para o tamanho de uma linha (chave + valor).
- `MAX_SECTIONS`: Número máximo de seções por arquivo (máximo 255).
- `MAX_ENTRIES`: Número máximo de entradas (chave-valor) no cache.
- `MAX_FILE_SIZE`: Tamanho máximo do nome do arquivo, incluindo o caminho.
- `PACK_CONTENT`: Se `true`, economiza memória, mas não suporta caracteres especiais.
- `USER_FILE_PATH`: Formato do caminho para arquivos de usuário (ex.: `Users/Player1.ini`).
- `USER_PW_HASH_KEY`: Chave usada para armazenar senhas hasheadas.

**Exemplo de Configuração Personalizada**:
```pawn
#define MAX_SECTION_TAG     (64)
#define MAX_LINE_SIZE       (200)
#define MAX_SECTIONS        (50)
#define MAX_ENTRIES         (500)
#define PACK_CONTENT        (false)
#define USER_FILE_PATH      "Players/%s.dat"
```

## Funções de Manipulação de Arquivos

### `DOF2_CreateFile(const file[], const password[] = "")`
Cria um novo arquivo INI. Se `password` for fornecido, armazena um hash na chave definida por `USER_PW_HASH_KEY`.
- **Parâmetros**:
  - `file`: Nome do arquivo (ex.: `Users/Player1.ini`).
  - `password`: Senha opcional.
- **Retorno**: `1` (sucesso), `0` (falha).
- **Exemplo**:
```pawn
DOF2_CreateFile("Users/Player1.ini", "senha123");
```

### `DOF2_SetFile(const file[])`
Define o arquivo atual para manipulação. Funções subsequentes (ex.: `DOF2_SetString`) usarão este arquivo.
- **Parâmetros**:
  - `file`: Nome do arquivo.
- **Exemplo**:
```pawn
DOF2_SetFile("Users/Player1.ini");
```

### `DOF2_SaveFile()`
Salva o arquivo atual no disco, gravando todas as alterações no cache.
- **Retorno**: `1` (sucesso), `0` (falha).
- **Exemplo**:
```pawn
DOF2_SetString("Users/Player1.ini", "Nome", "João", "Dados");
DOF2_SaveFile();
```

### `DOF2_FileExists(const file[])`
Verifica se um arquivo existe.
- **Parâmetros**:
  - `file`: Nome do arquivo.
- **Retorno**: `true` (existe), `false` (não existe).
- **Exemplo**:
```pawn
if (DOF2_FileExists("Users/Player1.ini")) {
    print("Arquivo encontrado!");
}
```

### `DOF2_RemoveFile(const file[])`
Remove um arquivo do sistema.
- **Parâmetros**:
  - `file`: Nome do arquivo.
- **Retorno**: `1` (sucesso), `0` (falha).
- **Exemplo**:
```pawn
DOF2_RemoveFile("Users/Player1.ini");
```

### `DOF2_RenameFile(const oldfile[], const newfile[])`
Renomeia um arquivo.
- **Parâmetros**:
  - `oldfile`: Nome atual do arquivo.
  - `newfile`: Novo nome do arquivo.
- **Retorno**: `1` (sucesso), `0` (falha).
- **Exemplo**:
```pawn
DOF2_RenameFile("Users/Player1.ini", "Users/Player2.ini");
```

### `DOF2_CopyFile(const filetocopy[], const newfile[])`
Copia um arquivo para um novo destino.
- **Parâmetros**:
  - `filetocopy`: Arquivo a ser copiado.
  - `newfile`: Nome do novo arquivo.
- **Retorno**: `1` (sucesso), `0` (falha).
- **Exemplo**:
```pawn
DOF2_CopyFile("Users/Player1.ini", "Users/Backup.ini");
```

### `DOF2_MakeBackup(const file[])`
Cria um backup do arquivo com nome baseado em data e hora.
- **Parâmetros**:
  - `file`: Nome do arquivo.
- **Retorno**: `1` (sucesso), `0` (falha).
- **Exemplo**:
```pawn
DOF2_MakeBackup("Users/Player1.ini"); // Cria algo como Users/Player1.07_23_2025.21_45_00_123.bak
```

## Funções de Manipulação de Entradas

### `DOF2_SetString(const file[], const key[], const value[], const tag[] = "")`
Define uma string em uma chave, opcionalmente em uma seção.
- **Parâmetros**:
  - `file`: Nome do arquivo.
  - `key`: Nome da chave.
  - `value`: Valor a ser armazenado.
  - `tag`: Seção (opcional, padrão é seção vazia).
- **Retorno**: `1` (sucesso), `0` (falha).
- **Exemplo**:
```pawn
DOF2_SetString("Users/Player1.ini", "Nome", "João Silva", "Dados");
```

### `DOF2_GetString(const file[], const key[], const tag[] = "")`
Obtém uma string de uma chave.
- **Parâmetros**:
  - `file`: Nome do arquivo.
  - `key`: Nome da chave.
  - `tag`: Seção (opcional).
- **Retorno**: String armazenada ou vazia se não encontrada.
- **Exemplo**:
```pawn
new nome[32];
nome = DOF2_GetString("Users/Player1.ini", "Nome", "Dados");
printf("Nome: %s", nome);
```

### `DOF2_SetInt(const file[], const key[], value, const tag[] = "")`
Define um valor inteiro.
- **Exemplo**:
```pawn
DOF2_SetInt("Users/Player1.ini", "Score", 1500, "Stats");
```

### `DOF2_GetInt(const file[], const key[], const tag[] = "")`
Obtém um valor inteiro.
- **Exemplo**:
```pawn
new score = DOF2_GetInt("Users/Player1.ini", "Score", "Stats");
printf("Score: %d", score);
```

### `DOF2_SetFloat(const file[], const key[], Float:value, const tag[] = "")`
Define um valor flutuante.
- **Exemplo**:
```pawn
DOF2_SetFloat("Users/Player1.ini", "PosX", 123.456, "Position");
```

### `DOF2_GetFloat(const file[], const key[], const tag[] = "")`
Obtém um valor flutuante.
- **Exemplo**:
```pawn
new Float:posX = DOF2_GetFloat("Users/Player1.ini", "PosX", "Position");
printf("PosX: %.2f", posX);
```

### `DOF2_SetBool(const file[], const key[], bool:value, const tag[] = "")`
Define um valor booleano.
- **Exemplo**:
```pawn
DOF2_SetBool("Users/Player1.ini", "IsAdmin", true, "Perms");
```

### `DOF2_GetBool(const file[], const key[], const tag[] = "")`
Obtém um valor booleano.
- **Exemplo**:
```pawn
if (DOF2_GetBool("Users/Player1.ini", "IsAdmin", "Perms")) {
    print("Jogador é admin!");
}
```

### `DOF2_SetHex(const file[], const key[], value, const tag[] = "")`
Define um valor hexadecimal.
- **Exemplo**:
```pawn
DOF2_SetHex("Users/Player1.ini", "Color", 0xFF0000FF, "Settings");
```

### `DOF2_GetHex(const file[], const key[], const tag[] = "")`
Obtém um valor hexadecimal.
- **Exemplo**:
```pawn
new color = DOF2_GetHex("Users/Player1.ini", "Color", "Settings");
printf("Cor: 0x%X", color);
```

### `DOF2_SetBin(const file[], const key[], value, const tag[] = "")`
Define um valor binário.
- **Exemplo**:
```pawn
DOF2_SetBin("Users/Player1.ini", "Flags", 0b1010, "Settings");
```

### `DOF2_GetBin(const file[], const key[], const tag[] = "")`
Obtém um valor binário.
- **Exemplo**:
```pawn
new flags = DOF2_GetBin("Users/Player1.ini", "Flags", "Settings");
printf("Flags: %b", flags);
```

### `DOF2_Unset(const file[], const key[], const tag[] = "")`
Remove uma chave.
- **Exemplo**:
```pawn
DOF2_Unset("Users/Player1.ini", "Nome", "Dados");
```

### `DOF2_IsSet(const file[], const key[], const tag[] = "")`
Verifica se uma chave existe.
- **Exemplo**:
```pawn
if (DOF2_IsSet("Users/Player1.ini", "Nome", "Dados")) {
    print("Chave existe!");
}
```

### `DOF2_RenameKey(const file[], oldkey[], newkey[], const tag[] = "")`
Renomeia uma chave.
- **Exemplo**:
```pawn
DOF2_RenameKey("Users/Player1.ini", "Nome", "Nick", "Dados");
```

## Funções de Manipulação de Seções

### `DOF2_SectionExists(const file[], const tag[])`
Verifica se uma seção existe.
- **Parâmetros**:
  - `file`: Nome do arquivo.
  - `tag`: Nome da seção.
- **Retorno**: `true` (existe), `false` (não existe).
- **Exemplo**:
```pawn
if (DOF2_SectionExists("Users/Player1.ini", "Dados")) {
    print("Seção Dados existe!");
}
```

### `DOF2_RemoveSection(const file[], const tag[])`
Remove uma seção e suas entradas.
- **Exemplo**:
```pawn
DOF2_RemoveSection("Users/Player1.ini", "Dados");
```

### `DOF2_SortSection(const file[], const tag[], bool:ignorecase = true, bool:ascending = true)`
Ordena as entradas de uma seção alfabeticamente.
- **Parâmetros**:
  - `file`: Nome do arquivo.
  - `tag`: Nome da seção.
  - `ignorecase`: Ignorar maiúsculas/minúsculas (`true` por padrão).
  - `ascending`: Ordem ascendente (`true`) ou descendente (`false`).
- **Exemplo**:
```pawn
DOF2_SortSection("Users/Player1.ini", "Stats", true, true);
```

### `DOF2_SortAllSections(const file[], bool:ignorecase = true, bool:ascending = true)`
Ordena todas as seções de um arquivo.
- **Exemplo**:
```pawn
DOF2_SortAllSections("Users/Player1.ini", true, true);
```

## Parsing e Callbacks

### `DOF2_ParseFile(const file[], extraid = -1, bool:callback = false)`
Carrega e analisa um arquivo INI, chamando callbacks se `callback = true`.
- **Parâmetros**:
  - `file`: Nome do arquivo.
  - `extraid`: ID extra para passar aos callbacks (ex.: `playerid`).
  - `callback`: Ativar (`true`) ou desativar (`false`) callbacks.
- **Retorno**: `1` (sucesso), `0` (falha).
- **Exemplo**:
```pawn
DOF2_ParseFile("Users/Player1.ini", playerid, true);
```

### `OnParseFile_<Tag>_<Key>(extraid, value[])`
Callback chamado para uma chave específica em uma seção.
- **Exemplo**:
```pawn
OnParseFile_Dados_Nome(playerid, value[]) {
    printf("Jogador %d tem nome: %s", playerid, value);
    return 1;
}
```

### `OnDefaultParseFile(extraid, value[], key[], tag[], file[])`
Callback padrão para entradas sem callback específico.
- **Exemplo**:
```pawn
OnDefaultParseFile(extraid, value[], key[], tag[], file[]) {
    printf("Arquivo: %s, Seção: %s, Chave: %s, Valor: %s", file, tag, key, value);
    return 1;
}
```

## Funções Auxiliares

### `DOF2_SetUTF8(bool:set)`
Ativa/desativa suporte a UTF-8. Se `false`, economiza memória, mas não suporta caracteres especiais.
- **Exemplo**:
```pawn
DOF2_SetUTF8(true);
```

### `DOF2_GetUTF8()`
Retorna o estado do suporte a UTF-8 (`true` ou `false`).
- **Exemplo**:
```pawn
if (DOF2_GetUTF8()) {
    print("Suporte a UTF-8 ativado!");
}
```

### `DOF2_SetCaseSensitivity(bool:set)`
Define se as chaves são sensíveis a maiúsculas/minúsculas.
- **Exemplo**:
```pawn
DOF2_SetCaseSensitivity(false);
```

### `DOF2_GetCaseSensitivity()`
Retorna o estado da sensibilidade a maiúsculas/minúsculas (`true` ou `false`).
- **Exemplo**:
```pawn
if (!DOF2_GetCaseSensitivity()) {
    print("Chaves não são sensíveis a maiúsculas/minúsculas!");
}
```

### `DOF2_CheckLogin(const file[], const password[])`
Verifica se a senha corresponde ao hash armazenado.
- **Exemplo**:
```pawn
if (DOF2_CheckLogin(DOF2_File("Player1"), "senha123")) {
    print("Login correto!");
}
```

### `DOF2_File(const user[])`
Gera o caminho do arquivo para um usuário, baseado em `USER_FILE_PATH`.
- **Exemplo**:
```pawn
new file[64];
file = DOF2_File("Player1"); // Retorna "Users/Player1.ini"
```

### `DOF2_PrintFile(const comment[] = "")`
Exibe o conteúdo do arquivo atual no console.
- **Exemplo**:
```pawn
DOF2_PrintFile("Exibindo dados do jogador");
```

## Macros Úteis
- `DOF2_LoadFile()`: Alias para `DOF2_ParseFile(CurrentFile, -1, false)`.
- `DOF2_SaveFile`: Alias para `DOF2_WriteFile`.
- `DOF2_FileExists`: Alias para `fexist`.
- `DOF2_ParseInt()`: Converte string para inteiro (`strval(value)`).
- `DOF2_ParseFloat()`: Converte string para float (`floatstr(value)`).
- `DOF2_ParseBool()`: Converte string para booleano (`strval(value) || !strcmp(value, "true", true)`).
- `DOF2_TagExists`: Alias para `DOF2_SectionExists`.
- `DOF2_RemoveTag`: Alias para `DOF2_RemoveSection`.

## Compatibilidade com DUDB e DINI
Ative a compatibilidade com:
```pawn
#define DUDB_CONVERT
#define DINI_CONVERT
#include <DOF2>
```

### Macros DUDB
- `dUser("Player1").("Nome")`: Equivale a `DOF2_GetString(DOF2_File("Player1"), "Nome")`.
- `dUserSet("Player1").("Nome", "João")`: Equivale a `DOF2_SetString(DOF2_File("Player1"), "Nome", "João")`.

### Macros DINI
- `dini_Set("Users/Player1.ini", "Nome", "João")`: Equivale a `DOF2_SetString`.
- `dini_Get("Users/Player1.ini", "Nome")`: Equivale a `DOF2_GetString`.

## Exemplo Completo de Sistema de Contas
Exemplo de sistema de contas de jogadores que salva e carrega dados:

```pawn
#include <a_samp>
#include <DOF2>

new PlayerData[MAX_PLAYERS][5]; // 0: Score, 1: Dinheiro, 2: Float:PosX, 3: Float:PosY, 4: bool:IsAdmin

public OnPlayerConnect(playerid) {
    new file[64], playerName[MAX_PLAYER_NAME];
    GetPlayerName(playerid, playerName, sizeof(playerName));
    format(file, sizeof(file), "Users/%s.ini", playerName);

    if (!DOF2_FileExists(file)) {
        // Novo jogador: criar arquivo e definir valores padrão
        DOF2_CreateFile(file, "senha123");
        DOF2_SetString(file, "Nome", playerName, "Dados");
        DOF2_SetInt(file, "Score", 0, "Stats");
        DOF2_SetInt(file, "Dinheiro", 1000, "Stats");
        DOF2_SetFloat(file, "PosX", 0.0, "Position");
        DOF2_SetFloat(file, "PosY", 0.0, "Position");
        DOF2_SetBool(file, "IsAdmin", false, "Perms");
        DOF2_SaveFile();
    } else {
        // Carregar dados do jogador
        DOF2_SetFile(file);
        PlayerData[playerid][0] = DOF2_GetInt(file, "Score", "Stats");
        PlayerData[playerid][1] = DOF2_GetInt(file, "Dinheiro", "Stats");
        PlayerData[playerid][2] = DOF2_GetFloat(file, "PosX", "Position");
        PlayerData[playerid][3] = DOF2_GetFloat(file, "PosY", "Position");
        PlayerData[playerid][4] = DOF2_GetBool(file, "IsAdmin", "Perms");
        printf("Bem-vindo %s! Score: %d, Dinheiro: %d, Pos: (%.2f, %.2f), Admin: %d",
            playerName, PlayerData[playerid][0], PlayerData[playerid][1],
            PlayerData[playerid][2], PlayerData[playerid][3], PlayerData[playerid][4]);
    }
    return 1;
}

public OnPlayerDisconnect(playerid, reason) {
    new file[64], playerName[MAX_PLAYER_NAME];
    GetPlayerName(playerid, playerName, sizeof(playerName));
    format(file, sizeof(file), "Users/%s.ini", playerName);

    DOF2_SetInt(file, "Score", PlayerData[playerid][0], "Stats");
    DOF2_SetInt(file, "Dinheiro", PlayerData[playerid][1], "Stats");
    DOF2_SetFloat(file, "PosX", PlayerData[playerid][2], "Position");
    DOF2_SetFloat(file, "PosY", PlayerData[playerid][3], "Position");
    DOF2_SetBool(file, "IsAdmin", PlayerData[playerid][4], "Perms");
    DOF2_SaveFile();
    return 1;
}
```

## Dicas e Boas Práticas
- Sempre chame `DOF2_SaveFile()` após modificar dados para garantir a persistência.
- Use seções para organizar dados (ex.: `[Stats]`, `[Position]`).
- Evite exceder `MAX_ENTRIES` ou `MAX_SECTIONS` para manter a performance.
- Ative `DOF2_SetUTF8(true)` para suportar caracteres especiais, se necessário.
- Use `DOF2_PrintFile()` para depuração de arquivos INI.
- Faça backups regulares com `DOF2_MakeBackup()`.

## Limitações
- Não suporta comentários em arquivos INI.
- O número de seções e entradas é limitado por `MAX_SECTIONS` e `MAX_ENTRIES`.
- Suporte a UTF-8 requer `PACK_CONTENT = false`, aumentando o uso de memória.

## Como Hospedar no GitHub
Para compartilhar esta documentação no GitHub:
1. Crie um repositório (ex.: `DOF2-Documentation`).
2. Salve este arquivo como `DOF2_Documentation.md` ou use-o como `README.md`.
3. Faça upload do arquivo para o repositório:
   - Via interface web: Clique em "Add file" > "Upload files".
   - Via Git:
     ```bash
     git init
     git add DOF2_Documentation.md
     git commit -m "Adiciona documentação do DOF2"
     git remote add origin <URL_DO_REPOSITORIO>
     git push -u origin main
     ```
4. Adicione uma descrição no repositório, como:
   ```
   Documentação completa do DOF2 para SA-MP, com funções, exemplos e boas práticas.
   ```
5. Compartilhe o link do repositório (ex.: `https://github.com/SEU_USUARIO/DOF2-Documentation`).

O conteúdo será renderizado automaticamente pelo GitHub, permitindo que qualquer IA ou usuário acesse e leia diretamente pelo link.