Desde 2013, essa função permaneceu secreta para a maioria dos scripters, sem documentação em lugar algum e por ninguém. Bem, é hora de lançar luz sobre esse lado obscuro do SAMP.

**SendClientCheck**

Essa função envia uma solicitação especial para o cliente, que então responde através do callback **OnClientCheckResponse**. A latência da resposta depende do ping do jogador.

A função e o callback não estão declarados nos includes, então você precisará declará-los por conta própria, por exemplo, no início do seu script:
```c
native SendClientCheck(playerid, type, arg, offset, size);
forward OnClientCheckResponse(playerid, type, arg, response);
```
Dê uma olhada nos argumentos de **SendClientCheck**:  
- **playerid** – qual jogador será solicitado  
- **type** – tipo de solicitação  
- **arg** – argumento especial  
- **offset** – deslocamento de leitura na memória  
- **size** – número de bytes a serem lidos (sempre deve ser maior ou igual a 2)  

E os argumentos de **OnClientCheckResponse**:  
- **playerid** – quem enviou a resposta  
- **type** – tipo de resposta  
- **arg** – argumento especial  
- **response** – outro argumento especial  

Agora, vamos ver o que tudo isso significa. Existem **6 tipos de solicitações** que o cliente processa (argumento **type**): **2, 5, 69, 70, 71, 72**.

### **Tipo nº 72**  
Vamos começar com uma solicitação simples. Esse tipo de solicitação **não usa nenhum argumento** (**arg, offset, size**). Ele retorna o **tempo de atividade do computador do jogador** dentro de **arg**.  

**Exemplo:**
```c
#include <a_samp>
native SendClientCheck(playerid, type, arg, offset, size); forward OnClientCheckResponse(playerid, type, arg, response);

public OnPlayerConnect(playerid)
{
        SendClientCheck(playerid, 72, 0, 0, 2);
        return 1;
}

public OnClientCheckResponse(playerid, type, arg, response)
{
        new str[128];
        switch(type)
        {
                case 72:
                {
                        format(str, sizeof(str), "Seu computador está funcionando há %s!", Convert(arg / 1000));
                        SendClientMessage(playerid, -1, str);
                }
        }
        return 1;
}

Convert(number)
{
        new hours = 0, mins = 0, secs = 0, string[100]; hours = floatround(number / 3600);
        mins = floatround((number / 60) - (hours * 60)); secs = floatround(number - ((hours * 3600) + (mins * 60)));
        if(hours > 0)
        {
                format(string, 100, "%d:%02d:%02d", hours, mins, secs);
        }
        else
        {
                format(string, 100, "%d:%02d", mins, secs);
        }
        return string;
}
```
Este exemplo exibirá o **tempo de atividade do computador do jogador** quando ele entrar no servidor.
---
### **Tipo nº 70**  
Este tipo **lê dados da estrutura CBaseModelInfoSA** de um modelo específico e retorna um **checksum em bytes**. É possível especificar **de qual deslocamento (offset) começar a leitura** e **quantos bytes ler**.  

Por exemplo, o comando:  
```c
SendClientCheck(playerid, 70, 1598, 0, 28);
```
fará uma solicitação que retornará o **checksum dos primeiros 28 bytes** da estrutura do modelo **1598**.  

Um exemplo mais completo:
```c
public OnPlayerConnect(playerid)
{
        SendClientCheck(playerid, 70, 1598, 0, 28); // 1598 - beachball
        return 1;
}

public OnClientCheckResponse(playerid, type, arg, response)
{
        new str[128];
        switch(type)
        {
                case 70:
                {
                        format(str, sizeof(str), "Modelo %d tem checksum 0x%x", arg, response);
                        SendClientMessage(playerid, -1, str);
                }
        }
        return 1;
}
```
---
### **Tipo nº 71**  
Este tipo **lê dados da estrutura CColModelSA** de um modelo específico (**cujo ponteiro está localizado no deslocamento 20 da estrutura CBaseModelInfoSA**). O restante funciona de maneira semelhante ao tipo anterior.  

**Exemplo:**  
```c
public OnPlayerConnect(playerid)
{
        SendClientCheck(playerid, 71, 1598, 0, 48); // 1598 - beachball
        return 1;
}

public OnClientCheckResponse(playerid, type, arg, response)
{
        new str[128];
        switch(type)
        {
                case 71:
                {
                        format(str, sizeof(str), "Col model %d tem checksum 0x%x", arg, response);
                        SendClientMessage(playerid, -1, str);
                }
        }
        return 1;
}
```
---
### **Tipo nº 2**  
Agora estamos chegando às partes interessantes. Esse tipo retorna **32 flags da estrutura CPhysicalSA**.  
- **Se o jogador estiver em um veículo**, retorna informações sobre o veículo.  
- **Se o jogador estiver a pé**, retorna informações sobre o próprio jogador.  

As flags incluem informações como:  
- Se o jogador está no chão  
- Se está na água  
- Se está tocando a água  

O exemplo de uso está disponível no link abaixo.  

### ⚠ **Atenção:**  
Os tipos abaixo funcionam **somente nas versões do cliente**:  
- **0.3.7-R2**  
- **0.3.7-R3**  
- **0.3.DL-R1**  
---
### **Tipo nº 5**  
Esse tipo gera um **checksum de `size` bytes** a partir do endereço **`arg + offset`** (memória do GTA) e retorna no argumento **response**.  

✅ **Endereços válidos:** `0x400000 - 0x856E00`  
✅ **Offsets válidos:** `0 - 255`  

Este é um dos tipos **mais interessantes e úteis**, pois pode ser usado para verificar se a **memória foi modificada** por algum **cheat/hack**.  
---
### **Tipo nº 69**  
Esse tipo gera um **checksum de `size` bytes** a partir do endereço **`arg + offset`** (memória do SAMP) e retorna no argumento **response**.  

✅ **Endereços válidos:** `0x0 - 0xC3500`  
✅ **Offsets válidos:** `0 - 255`  

Assim como o tipo nº 5, também pode ser usado para **detectar cheats/hacks instalados**.  

### **Algoritmo de Checksum**  
O algoritmo é bem simples e funciona assim (pseudocódigo):  
```c
for(new i = startaddress; i < endaddress; i++)
{
        result ^= ReadMemory(i) & 0xCC;
}
```

Créditos: pawn.wiki
Traduzido por IA.
