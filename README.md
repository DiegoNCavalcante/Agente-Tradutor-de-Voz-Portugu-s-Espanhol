# 🌐 Agente Tradutor de Voz: Português → Espanhol

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/18mD-_owUKyijwbnZeoV7zotYCkHP7qeA?usp=sharing)

---

## Etapa 1 — Documentação do Agente

### Nome do Agente
**VozTradutor** — Agente de tradução de voz em tempo real.

### Descrição
O VozTradutor é um agente de inteligência artificial que capta a fala do usuário em português, transcreve o áudio, traduz o conteúdo para o espanhol e responde em voz alta. Ele combina três tecnologias da OpenAI e do Google para entregar uma experiência fluida e acessível.

### Objetivo
Permitir que qualquer pessoa se comunique em espanhol sem precisar digitar nada, apenas falando naturalmente em português.

### Público-alvo
- Estudantes de espanhol
- Viajantes
- Profissionais que trabalham com países hispânicos

---

## Etapa 2 — Base de Conhecimento

### Tecnologias utilizadas

| Tecnologia | Papel no Agente |
|---|---|
| OpenAI Whisper | Converte o áudio em texto (Speech-to-Text) |
| OpenAI ChatGPT | Traduz o texto de PT para ES |
| Google gTTS | Converte a tradução em áudio (Text-to-Speech) |
| Google Colab | Ambiente de execução na nuvem |

### Fontes de conhecimento
- O agente não possui base de dados própria
- O conhecimento de tradução vem do modelo **gpt-3.5-turbo** da OpenAI
- A transcrição é feita pelo modelo **whisper-1** da OpenAI

---

## Etapa 3 — Prompts do Agente

### System Prompt (instrução do agente)
```
Você é um tradutor especialista.
Traduza o texto do usuário do português para o espanhol.
Responda APENAS com a tradução, sem explicações adicionais.
```

### Exemplo de interação

| Entrada (usuário) | Saída (agente) |
|---|---|
| "Bom dia, como vai você?" | "Buenos días, ¿cómo estás?" |
| "Onde fica o banheiro?" | "¿Dónde está el baño?" |
| "Eu gostaria de um café, por favor." | "Me gustaría un café, por favor." |

---

## Etapa 4 — Aplicação Funcional

### Fluxo do agente

```
🎙️ Usuário fala em Português
        ↓
🔍 Whisper transcreve o áudio para texto
        ↓
🌍 ChatGPT traduz o texto para Espanhol
        ↓
🔊 gTTS converte a tradução em áudio
        ↓
✅ Usuário ouve a resposta em Espanhol
```

### Código principal

```python
# Instalar dependências
!pip install openai gtts -q

import openai
from gtts import gTTS
from IPython.display import display, Audio
from google.colab import output
import base64

openai.api_key = "SUA_CHAVE_AQUI"

# 1. Gravar áudio
def gravar_audio():
    js = """
    const sleep = ms => new Promise(res => setTimeout(res, ms));
    var audio, stream;
    async function main() {
        stream = await navigator.mediaDevices.getUserMedia({ audio: true });
        audio = new MediaRecorder(stream);
        var chunks = [];
        audio.ondataavailable = e => chunks.push(e.data);
        audio.start();
        await sleep(5000);
        audio.stop();
        stream.getTracks().forEach(t => t.stop());
        await sleep(500);
        var blob = new Blob(chunks);
        var reader = new FileReader();
        reader.readAsDataURL(blob);
        await new Promise(res => reader.onloadend = res);
        return reader.result.split(',')[1];
    }
    main();
    """
    audio_b64 = output.eval_js(js)
    audio_bytes = base64.b64decode(audio_b64)
    with open("audio.wav", "wb") as f:
        f.write(audio_bytes)

# 2. Transcrever com Whisper
def transcrever():
    with open("audio.wav", "rb") as f:
        resultado = openai.audio.transcriptions.create(
            model="whisper-1", file=f, language="pt"
        )
    return resultado.text

# 3. Traduzir com ChatGPT
def traduzir(texto):
    resposta = openai.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "Traduza do português para o espanhol. Responda APENAS com a tradução."},
            {"role": "user", "content": texto}
        ]
    )
    return resposta.choices[0].message.content.strip()

# 4. Falar com gTTS
def falar(texto):
    tts = gTTS(text=texto, lang="es")
    tts.save("traducao.mp3")
    display(Audio("traducao.mp3", autoplay=True))

# Executar
gravar_audio()
texto_pt = transcrever()
print(f"Você disse: {texto_pt}")
texto_es = traduzir(texto_pt)
print(f"Tradução: {texto_es}")
falar(texto_es)
```

---

## Etapa 5 — Avaliação e Métricas

### Critérios de avaliação

| Métrica | Como medir |
|---|---|
| Precisão da transcrição | Comparar texto transcrito com o que foi falado |
| Qualidade da tradução | Verificar se a tradução faz sentido contextualmente |
| Tempo de resposta | Medir tempo entre falar e ouvir a tradução |
| Naturalidade do áudio | Avaliar se a voz gerada soa natural |

### Resultados esperados
- Transcrição com **precisão > 90%** em ambiente silencioso
- Tradução **contextualmente correta** para frases do cotidiano
- Tempo de resposta entre **5 a 10 segundos**

---

## Etapa 6 — Pitch

### Problema
Muitas pessoas têm dificuldade de se comunicar em espanhol em situações do dia a dia, como viagens, reuniões ou atendimentos, sem ter tempo de digitar ou consultar um tradutor manualmente.

### Solução
O **VozTradutor** elimina essa barreira: basta falar em português e o agente responde instantaneamente em espanhol, por voz.

### Diferenciais
- ✅ 100% por voz — sem necessidade de digitar
- ✅ Roda na nuvem via Google Colab — sem instalação
- ✅ Usa modelos de ponta da OpenAI
- ✅ Simples e acessível para qualquer pessoa

### Próximos passos
- Suporte a mais idiomas (inglês, francês, alemão)
- Interface gráfica amigável
- Detecção automática do idioma falado

---

## 📝 Referências

- [OpenAI Whisper](https://platform.openai.com/docs/guides/speech-to-text)
- [OpenAI ChatGPT API](https://platform.openai.com/docs/guides/chat)
- [Google Text-to-Speech (gTTS)](https://gtts.readthedocs.io/)
- [Desafio DIO](https://bit.ly/41XfKaM)
