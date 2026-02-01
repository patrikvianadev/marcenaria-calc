# 📐 Calculadora de Furos - Documentação Técnica

> **Guia completo para desenvolvedores que desejam entender, modificar ou expandir este projeto usando IA (Claude, ChatGPT, etc.)**

---

## 📑 Índice

- [Visão Geral do Projeto](#-visão-geral-do-projeto)
- [Arquitetura e Estrutura](#-arquitetura-e-estrutura)
- [Stack Tecnológico](#-stack-tecnológico)
- [Lógica de Negócio](#-lógica-de-negócio)
- [Sistema de Validação](#-sistema-de-validação)
- [Visualização 2D e 3D](#-visualização-2d-e-3d)
- [PWA e Service Worker](#-pwa-e-service-worker)
- [Como Trabalhar com IA](#-como-trabalhar-com-ia-neste-projeto)
- [Roadmap e Melhorias](#-roadmap-e-melhorias-futuras)
- [Deploy e Build](#-deploy-e-build)
- [Troubleshooting](#-troubleshooting)

---

## 🎯 Visão Geral do Projeto

### O que é?

Uma Progressive Web App (PWA) que calcula posições exatas de furos em laterais de móveis para instalação de prateleiras. Desenvolvida para marceneiros profissionais e entusiastas.

### Problema Resolvido

- **Antes:** Marceneiros calculavam manualmente as alturas dos furos, sujeito a erros que causavam prateleiras tortas
- **Depois:** Cálculo automático, preciso e visualização antes da execução

### Características Principais

- ✅ **Zero dependências de build** - Funciona diretamente no navegador
- ✅ **Single-file architecture** - Todo código em um único HTML
- ✅ **Offline-first** - Service Worker com cache completo
- ✅ **Mobile-optimized** - Responsive design com touch support
- ✅ **3D visualization** - Three.js para visualização interativa
- ✅ **Real-time validation** - Validação enquanto usuário digita
- ✅ **Export capabilities** - PDF, print, clipboard

---

## 📁 Arquitetura e Estrutura

### Estrutura de Arquivos

```
calculadora-furos/
│
├── index.html              # Aplicação principal (1730+ linhas)
│   ├── <head>              # Meta tags, PWA config, CDN imports
│   ├── <style>             # CSS completo (responsive, print)
│   └── <script>            # JavaScript (validação, cálculo, 3D)
│
├── manifest.json           # PWA manifest (656 bytes)
│   ├── name                # "Calculadora de Furos"
│   ├── icons               # 192x192, 512x512
│   ├── start_url           # index.html
│   ├── display             # standalone
│   └── theme_color         # #4CAF50
│
├── sw.js                   # Service Worker (2.8KB)
│   ├── CACHE_NAME          # Versão do cache
│   ├── urlsToCache         # Assets para cache
│   ├── fetch handler       # Network-first strategy
│   └── activate handler    # Limpeza de caches antigos
│
├── gerar-icones.html       # Utilitário para gerar ícones PNG
│   └── Canvas API          # Desenha ícones 192x192 e 512x512
│
├── README.md               # Este arquivo (documentação técnica)
└── README-USUARIO.md       # Manual do usuário final
```

### Arquitetura de Single-File

Todo o código está em [index.html](index.html) dividido em 3 seções:

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- PWA meta tags, CDN imports -->
  </head>
  <body>
    <style>
      /* CSS completo (~400 linhas) */
    </style>

    <div class="container">
      <!-- HTML structure (~200 linhas) -->
    </div>

    <script>
      /* JavaScript completo (~1100 linhas) */
    </script>
  </body>
</html>
```

**Por que single-file?**
- ✅ Deploy simplificado (um único arquivo)
- ✅ Fácil de compartilhar e versionar
- ✅ Zero configuração de build tools
- ✅ Funciona offline imediatamente

---

## 🛠️ Stack Tecnológico

### Frontend

| Tecnologia | Versão | Uso | Justificativa |
|-----------|--------|-----|---------------|
| **HTML5** | - | Estrutura | Semântico, acessível |
| **CSS3** | - | Estilo + Layout | Flexbox, Grid, Media Queries |
| **JavaScript (Vanilla)** | ES6+ | Lógica | Zero overhead, performance |
| **Three.js** | r128 | Visualização 3D | WebGL rendering |
| **OrbitControls.js** | - | Controle de câmera 3D | Interação touch/mouse |
| **html2canvas** | 1.4.1 | Captura de tela | Geração de PDF |
| **jsPDF** | 2.5.1 | Export PDF | Download de documentos |

### PWA APIs

| API | Uso | Suporte |
|-----|-----|---------|
| **Service Worker** | Cache offline | 95%+ browsers |
| **Web App Manifest** | Instalação como app | 94%+ browsers |
| **Wake Lock API** | Manter tela ligada | 85%+ browsers |
| **Clipboard API** | Copiar medidas | 95%+ browsers |

### CDN Links

```html
<!-- Three.js - 3D rendering -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>

<!-- PDF generation -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
```

---

## 🧮 Lógica de Negócio

### Constantes Fundamentais

```javascript
const OFFSET_PARAFUSO = 4; // mm - Distância do parafuso à base da prateleira
```

**Por que 4mm?**
- Parafusos de prateleira (cavilhas, suportes) ficam ligeiramente acima da base
- Garante encaixe estável sem rasgar a madeira
- Padrão da indústria moveleira brasileira

### Fórmula de Cálculo

#### Primeira Prateleira:
```javascript
posicao = rodape + espessura_base + vao_1 + OFFSET_PARAFUSO
```

**Exemplo:**
```
Rodapé: 100mm
Base: 18mm
Vão 1: 300mm
Offset: 4mm
------------------
Resultado: 422mm
```

#### Prateleiras Subsequentes:
```javascript
posicao = posicao_anterior + (espessura - OFFSET_PARAFUSO) + vao_atual + OFFSET_PARAFUSO
```

**Exemplo (Prateleira 2):**
```
Posição anterior: 422mm
Espessura restante: 14mm (18mm - 4mm já usado)
Vão 2: 300mm
Offset: 4mm
------------------
Resultado: 740mm
```

### Fluxo de Cálculo

```javascript
function calcular() {
  // 1. Captura de inputs
  const espessura = parseFloat(document.getElementById('espessura').value);
  const rodape = temRodape ? parseFloat(document.getElementById('rodape').value) : 0;
  const vaos = Array.from(document.querySelectorAll('.vao-input')).map(input => parseFloat(input.value));

  // 2. Validação
  if (vaos.some(v => isNaN(v) || v <= 0)) {
    alert('Valores inválidos!');
    return;
  }

  // 3. Loop de cálculo
  const posicoes = [];
  let acumulado = rodape + espessura; // Altura inicial (base)

  for (let i = 0; i < vaos.length; i++) {
    acumulado += vaos[i] + OFFSET_PARAFUSO;
    posicoes.push({
      prateleira: i + 1,
      posicao: acumulado,
      calculo: gerarFormula(i, rodape, espessura, vaos, posicoes)
    });

    // Adiciona espessura para próxima iteração
    if (i < vaos.length - 1) {
      acumulado += (espessura - OFFSET_PARAFUSO);
    }
  }

  // 4. Renderização
  renderResultados(posicoes);
  renderVisualizacao2D(posicoes);
  preparaDados3D(posicoes);
}
```

### Cálculo Automático de Vãos

```javascript
function calcularAutomatico() {
  // Altura disponível = lateral - rodapé - base - topo
  const alturaDisponivel = lateral - rodape - (2 * espessura);

  // Altura ocupada pelas prateleiras intermediárias
  const alturaPreteleiras = (numVaos - 1) * espessura;

  // Altura livre para os vãos
  const alturaVaos = alturaDisponivel - alturaPreteleiras;

  // Divide igualmente
  const vaoAutomatico = Math.floor(alturaVaos / numVaos);

  // Aplica a todos os inputs
  inputs.forEach(input => input.value = vaoAutomatico);
}
```

---

## ✅ Sistema de Validação

### Validação em Tempo Real

```javascript
// Event listeners para validação instantânea
document.getElementById('lateral').addEventListener('input', validarConfiguracao);
document.getElementById('espessura').addEventListener('input', validarConfiguracao);
document.getElementById('vaosContainer').addEventListener('input', validarConfiguracao);
```

### Regras de Validação

| Regra | Tipo | Condição | Mensagem |
|-------|------|----------|----------|
| **Altura mínima** | Aviso | `lateral < 500mm` | "Altura muito baixa" |
| **Altura máxima** | Aviso | `lateral > 3000mm` | "Altura muito alta. Considere reforços" |
| **Espessura fina** | Aviso | `espessura < 15mm` | "Espessura muito fina. Recomendado: 15-25mm" |
| **Espessura grossa** | Aviso | `espessura > 30mm` | "Espessura acima do comum" |
| **Vão grande** | Aviso | `vao > 650mm && vao <= 800mm` | "Vão grande. Considere reforço" |
| **Vão muito grande** | Erro | `vao > 800mm` | "RISCO de vergamento! Máximo: 800mm" |
| **Altura excedida** | Erro | `alturaTotal > lateral` | "Altura total excede lateral" |
| **Rodapé alto** | Aviso | `rodape > lateral * 0.15` | "Rodapé alto. Verifique" |
| **Configuração OK** | Sucesso | Sem erros | "Configuração válida!" |

### Classes CSS de Validação

```css
.input-invalido {
  border-color: #f44336 !important;
  background: #ffebee !important;
}

.input-aviso {
  border-color: #ff9800 !important;
  background: #fff3e0 !important;
}

.validacao-erro {
  background: #ffebee;
  border-left-color: #f44336;
  color: #c62828;
}

.validacao-aviso {
  background: #fff3e0;
  border-left-color: #ff9800;
  color: #e65100;
}

.validacao-ok {
  background: #e8f5e9;
  border-left-color: #4CAF50;
  color: #2e7d32;
}
```

---

## 🎨 Visualização 2D e 3D

### Visualização 2D (Canvas HTML/CSS)

#### Estrutura HTML Gerada:

```html
<div class="visual">
  <h3>📦 Visualização do Armário</h3>

  <!-- Controles de zoom -->
  <div class="visual-controls">
    <button onclick="ajustarZoom(-0.25)">−</button>
    <div class="zoom-level">Zoom: <span id="zoomLevel">100%</span></div>
    <button onclick="ajustarZoom(0.25)">+</button>
  </div>

  <div class="visualizacao-wrapper">
    <!-- Régua lateral -->
    <div class="regua">
      <div class="regua-marca" style="top: 0px;">
        <span>2000mm</span>
        <div class="regua-linha"></div>
      </div>
      <!-- Mais marcas a cada 200mm -->
    </div>

    <!-- Armário -->
    <div class="armario" style="height: 250px;">
      <div class="rodape" style="height: 12.5px;"></div>
      <div class="base" style="bottom: 12.5px; height: 2.25px;"></div>
      <div class="prateleira" style="bottom: 52.75px; height: 2.25px;"></div>
      <div class="furo" style="bottom: 52.75px; left: 15%;"></div>
      <div class="furo" style="bottom: 52.75px; left: 85%;"></div>
      <!-- Mais prateleiras e furos -->
    </div>

    <!-- Legenda -->
    <div class="legenda">
      <div>🟫 Rodapé</div>
      <div>🟦 Prateleiras</div>
      <div>🔴 Furos</div>
    </div>
  </div>
</div>
```

#### Escala de Renderização:

```javascript
const escala = 250 / lateral; // Sempre cabe em 250px de altura
const alturaDiv = lateral * escala;
const rodapeVisual = rodape * escala;
const espessuraVisual = espessura * escala;
```

#### Sistema de Zoom:

```javascript
let zoomAtual = 1;

function ajustarZoom(delta) {
  zoomAtual += delta;
  zoomAtual = Math.max(0.5, Math.min(2, zoomAtual)); // Limita entre 50% e 200%

  document.querySelector('.armario').style.transform = `scale(${zoomAtual})`;
  document.getElementById('zoomLevel').textContent = `${Math.round(zoomAtual * 100)}%`;
}
```

### Visualização 3D (Three.js)

#### Setup da Cena:

```javascript
function renderizar3D(lateral, rodape, espessura, vaos, posicoes, temRodape) {
  // 1. Scene setup
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0xf0f0f0);
  scene.fog = new THREE.Fog(0xf0f0f0, 500, 2000);

  // 2. Camera setup (Perspective)
  const aspect = container.offsetWidth / container.offsetHeight;
  const camera = new THREE.PerspectiveCamera(45, aspect, 1, 5000);
  camera.position.set(lateral * 1.5, lateral * 0.8, lateral * 1.5);
  camera.lookAt(0, lateral / 2, 0);

  // 3. Renderer setup (WebGL)
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(container.offsetWidth, container.offsetHeight);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;

  // 4. OrbitControls (interação)
  const controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;
  controls.minDistance = lateral * 0.5;
  controls.maxDistance = lateral * 3;

  // 5. Lighting
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
  directionalLight.position.set(lateral, lateral * 2, lateral);
  directionalLight.castShadow = true;

  // 6. Grid floor (referência)
  const gridHelper = new THREE.GridHelper(lateral * 2, 20, 0x888888, 0xcccccc);
  scene.add(gridHelper);

  // 7. Geometrias do armário
  criarLaterais(scene, lateral, espessura);
  criarRodape(scene, lateral, rodape, espessura);
  criarPrateleiras(scene, lateral, espessura, posicoes);
  criarFuros(scene, lateral, posicoes);

  // 8. Animation loop
  function animate() {
    requestAnimationFrame(animate);
    controls.update();
    renderer.render(scene, camera);
  }
  animate();
}
```

#### Materiais:

```javascript
// Material das prateleiras (azul)
const materialPrateleira = new THREE.MeshStandardMaterial({
  color: 0x2196F3,
  metalness: 0.1,
  roughness: 0.6
});

// Material das laterais (madeira clara)
const materialLateral = new THREE.MeshStandardMaterial({
  color: 0xD2B48C,
  metalness: 0.0,
  roughness: 0.8
});

// Material dos furos (vermelho)
const materialFuro = new THREE.MeshStandardMaterial({
  color: 0xf44336,
  metalness: 0.5,
  roughness: 0.3
});
```

#### Criação de Prateleiras:

```javascript
function criarPrateleiras(scene, lateral, espessura, posicoes) {
  const largura = 400;
  const profundidade = 300;

  posicoes.forEach(p => {
    const geometry = new THREE.BoxGeometry(largura, espessura, profundidade);
    const mesh = new THREE.Mesh(geometry, materialPrateleira);
    mesh.position.set(0, p.posicao - OFFSET_PARAFUSO, 0);
    mesh.castShadow = true;
    mesh.receiveShadow = true;
    scene.add(mesh);

    // 4 furos por prateleira (2 esquerda, 2 direita)
    criarFurosPrateleira(scene, p.posicao, largura, profundidade);
  });
}
```

---

## 📱 PWA e Service Worker

### Web App Manifest

```json
{
  "name": "Calculadora de Furos",
  "short_name": "Calc Furos",
  "description": "Calculadora profissional de furos para prateleiras",
  "start_url": "index.html",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#4CAF50",
  "orientation": "any",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
}
```

### Service Worker Strategy

**Network-first com fallback para cache:**

```javascript
self.addEventListener('fetch', event => {
  event.respondWith(
    fetch(event.request)
      .then(response => {
        // Se conseguiu da rede, atualiza o cache
        const responseToCache = response.clone();
        caches.open(CACHE_NAME).then(cache => {
          cache.put(event.request, responseToCache);
        });
        return response;
      })
      .catch(() => {
        // Se falhou, busca do cache
        return caches.match(event.request);
      })
  );
});
```

**Por que Network-first?**
- ✅ Sempre tenta buscar a versão mais recente
- ✅ Funciona offline se já foi acessado antes
- ✅ Atualiza automaticamente quando tem internet

### Wake Lock API

```javascript
let wakeLock = null;

async function requestWakeLock() {
  try {
    if ('wakeLock' in navigator) {
      wakeLock = await navigator.wakeLock.request('screen');
      console.log('Wake Lock ativado');
    }
  } catch (err) {
    console.log('Wake Lock não disponível:', err);
  }
}

// Reativa quando a página volta a ser visível
document.addEventListener('visibilitychange', async () => {
  if (wakeLock !== null && document.visibilityState === 'visible') {
    await requestWakeLock();
  }
});
```

---

## 🤖 Como Trabalhar com IA neste Projeto

### Prompts Eficazes para Claude/ChatGPT

#### 1. **Entender a Base de Código**

```
Analise o arquivo index.html deste projeto de calculadora de furos.
Identifique:
- Estrutura geral do código
- Principais funções e suas responsabilidades
- Dependências externas
- Padrões de design utilizados
```

#### 2. **Adicionar Nova Funcionalidade**

```
Preciso adicionar [FUNCIONALIDADE] neste projeto.

Contexto:
- O projeto é uma PWA single-file
- Todo código está em index.html
- Usa Three.js para 3D, html2canvas para PDF
- Validação em tempo real já existe

Requisitos:
1. Manter compatibilidade com código existente
2. Seguir padrões de validação já implementados
3. Adicionar testes manuais no final

Por favor, mostre:
- Onde adicionar o código (linha aproximada)
- Código completo da funcionalidade
- Como testar manualmente
```

#### 3. **Refatorar Código Existente**

```
Quero refatorar a função [NOME_FUNCAO] no index.html.

Objetivos:
- Melhorar legibilidade
- Manter mesma funcionalidade
- Não quebrar integrações existentes

Mostre:
- Código original
- Código refatorado
- O que mudou e por quê
- Possíveis side effects
```

#### 4. **Corrigir Bug**

```
Há um bug em [FUNCIONALIDADE]:
[Descrever o comportamento esperado vs atual]

Arquivo: index.html, aproximadamente linha [LINHA]

Ajude-me a:
1. Identificar a causa raiz
2. Propor solução
3. Testar que não quebrou nada
```

#### 5. **Adicionar Testes**

```
Preciso criar testes para a função calcular() do projeto.

Contexto:
- Projeto não tem framework de testes ainda
- Pode sugerir Jest, Mocha ou vanilla JS

Crie:
1. Setup de testes
2. Casos de teste (positivos e negativos)
3. Como rodar os testes
```

### Estrutura de Arquivos Sugerida para Expansão

Se o projeto crescer, sugira esta estrutura ao AI:

```
calculadora-furos/
├── src/
│   ├── core/
│   │   ├── calculos.js        # Lógica de cálculo pura
│   │   ├── validacoes.js      # Regras de validação
│   │   └── templates.js       # Templates prontos
│   ├── ui/
│   │   ├── forms.js           # Manipulação de formulários
│   │   ├── visual2d.js        # Renderização 2D
│   │   └── visual3d.js        # Renderização 3D
│   ├── services/
│   │   ├── pdfExport.js       # Geração de PDF
│   │   ├── wakeLock.js        # Wake Lock API
│   │   └── clipboard.js       # Copiar para clipboard
│   └── utils/
│       ├── dom.js             # Helpers DOM
│       └── math.js            # Funções matemáticas
├── styles/
│   ├── main.css               # Estilos base
│   ├── responsive.css         # Media queries
│   └── print.css              # Estilos de impressão
├── tests/
│   ├── calculos.test.js
│   ├── validacoes.test.js
│   └── integration.test.js
├── public/
│   ├── index.html
│   ├── manifest.json
│   └── sw.js
└── dist/                      # Build output
```

### Perguntas Importantes para Fazer ao AI

Antes de pedir mudanças, pergunte:

1. **"Qual o impacto desta mudança nas funcionalidades existentes?"**
2. **"Esta mudança requer alteração no Service Worker?"**
3. **"Como isso afeta a experiência mobile?"**
4. **"Preciso atualizar o manifest.json?"**
5. **"Esta biblioteca adiciona quanto de tamanho ao app?"**
6. **"Funciona offline?"**
7. **"É compatível com navegadores antigos?"**

### Exemplo Completo de Prompt

```
Contexto:
Estou trabalhando na "Calculadora de Furos", uma PWA single-file
para marceneiros. Todo código está em index.html (1730 linhas).

Objetivo:
Adicionar funcionalidade de salvar projetos no LocalStorage.

Requisitos:
1. Botão "💾 Salvar Projeto" ao lado do botão calcular
2. Campo de texto para nome do projeto
3. Lista de projetos salvos (dropdown select)
4. Botão "📂 Carregar Projeto" que preenche os inputs
5. Botão "🗑️ Deletar Projeto" ao lado de cada item
6. Usar LocalStorage (não IndexedDB, manter simples)
7. JSON format para armazenar: {nome, lateral, espessura, rodape, vaos}

Restrições:
- Não mudar a estrutura HTML existente drasticamente
- Manter estilo consistente (verde #4CAF50)
- Funcionar offline
- Mobile-friendly

Por favor, forneça:
1. HTML dos novos elementos (onde inserir no código)
2. CSS para os novos componentes (dentro da tag <style>)
3. JavaScript das funções (onde no <script>)
4. Instruções de teste

Formato de resposta:
- Marque claramente onde inserir cada bloco de código
- Numere as linhas aproximadas
- Explique o que cada função faz
```

---

## 🚀 Roadmap e Melhorias Futuras

### Prioridade Alta (Impacto Imediato)

#### 1. **Salvar Projetos Localmente**
```javascript
// LocalStorage para salvar configurações
const projeto = {
  nome: "Estante Sala",
  timestamp: Date.now(),
  config: { lateral, espessura, rodape, vaos }
};
localStorage.setItem(`projeto_${projeto.nome}`, JSON.stringify(projeto));
```

**Benefícios:**
- Usuário não perde trabalho ao fechar o app
- Pode ter múltiplos projetos salvos
- Facilita comparação entre diferentes configurações

#### 2. **Modo Escuro**
```css
@media (prefers-color-scheme: dark) {
  body { background: #1e1e1e; color: #ffffff; }
  .container { background: #2d2d2d; }
  input { background: #3d3d3d; color: #ffffff; border-color: #555; }
}
```

**Benefícios:**
- Conforto visual em ambientes escuros
- Economia de bateria (OLED screens)
- Preferência de 40%+ dos usuários mobile

#### 3. **Exportar como Imagem PNG**
```javascript
function exportarImagem() {
  const elemento = document.querySelector('.visual');
  html2canvas(elemento, {
    scale: 2, // Alta resolução
    backgroundColor: '#ffffff'
  }).then(canvas => {
    const link = document.createElement('a');
    link.download = `armario_${Date.now()}.png`;
    link.href = canvas.toDataURL('image/png');
    link.click();
  });
}
```

**Benefícios:**
- Mais rápido que PDF
- Fácil de compartilhar no WhatsApp
- Menor tamanho de arquivo

#### 4. **Múltiplos Parafusos por Prateleira**
```javascript
// Adicionar configuração de quantidade de parafusos
const numParafusos = document.getElementById('numParafusos').value; // 2, 3 ou 4
// Calcular posições horizontais (distribuição uniforme)
const espacamento = largura / (numParafusos + 1);
for (let i = 1; i <= numParafusos; i++) {
  const posX = espacamento * i;
  criarFuro(posX, posY);
}
```

**Benefícios:**
- Maior estabilidade para prateleiras longas
- Customização para diferentes tipos de madeira

### Prioridade Média (UX Improvements)

#### 5. **Histórico de Cálculos**
```javascript
const historico = JSON.parse(localStorage.getItem('historico') || '[]');
historico.unshift({
  timestamp: new Date().toISOString(),
  config: { lateral, espessura, rodape, vaos },
  resultados: posicoes
});
localStorage.setItem('historico', JSON.stringify(historico.slice(0, 10))); // Mantém últimos 10
```

#### 6. **Unidades de Medida Alternativas**
```javascript
const unidades = {
  mm: { label: 'Milímetros', multiplicador: 1 },
  cm: { label: 'Centímetros', multiplicador: 10 },
  in: { label: 'Polegadas', multiplicador: 25.4 }
};

function converterUnidade(valor, de, para) {
  return (valor * unidades[de].multiplicador) / unidades[para].multiplicador;
}
```

#### 7. **Sugestões Inteligentes**
```javascript
// Análise heurística
function sugerirMelhorias(config) {
  const sugestoes = [];

  if (config.vaoMedio > 600 && config.espessura < 25) {
    sugestoes.push('💡 Considere usar prateleira de 25mm para maior resistência');
  }

  if (config.sobra > 150) {
    sugestoes.push('💡 Há espaço para adicionar mais uma prateleira');
  }

  return sugestoes;
}
```

#### 8. **Tutorial Interativo (First-Time User Experience)**
```javascript
if (!localStorage.getItem('tutorial_completo')) {
  mostrarTutorial([
    { elemento: '#lateral', texto: 'Comece inserindo a altura da lateral' },
    { elemento: '#espessura', texto: 'Depois a espessura das prateleiras' },
    { elemento: '.btn-template', texto: 'Ou use um template pronto!' }
  ]);
}
```

### Prioridade Baixa (Nice to Have)

#### 9. **Múltiplas Línguas (i18n)**
```javascript
const traducoes = {
  'pt-BR': { titulo: 'Calculadora de Furos', calcular: 'Calcular' },
  'en-US': { titulo: 'Hole Calculator', calcular: 'Calculate' },
  'es-ES': { titulo: 'Calculadora de Agujeros', calcular: 'Calcular' }
};

function t(chave) {
  const idioma = localStorage.getItem('idioma') || navigator.language;
  return traducoes[idioma][chave] || traducoes['pt-BR'][chave];
}
```

#### 10. **Integração com Loja Online**
```javascript
// Link para comprar materiais
function gerarListaMateriais() {
  return {
    prateleiras: `${numPrateleiras}x MDF ${espessura}mm 400x300mm`,
    parafusos: `${numPrateleiras * 4}x Cavilha 8mm`,
    laterais: `2x MDF ${espessura}mm ${lateral}x400mm`
  };
}

function buscarPrecosMLB(materiais) {
  // Integração com API do Mercado Livre
}
```

#### 11. **Compartilhamento Social**
```javascript
async function compartilharWhatsApp() {
  const texto = `📐 Projeto de Armário
Altura: ${lateral}mm
Prateleiras: ${vaos.length}
Veja o cálculo completo: ${window.location.href}`;

  const url = `https://wa.me/?text=${encodeURIComponent(texto)}`;
  window.open(url, '_blank');
}
```

#### 12. **Cálculo de Peso Suportado**
```javascript
function calcularPesoMaximo(espessura, vao, materialTipo) {
  const resistencias = {
    'MDF': { 15: 5, 18: 8, 25: 15 },      // kg/m
    'Compensado': { 15: 8, 18: 12, 25: 20 },
    'Madeira': { 15: 10, 18: 15, 25: 25 }
  };

  const resistencia = resistencias[materialTipo][espessura];
  const comprimento = vao / 1000; // mm para metros
  return Math.floor(resistencia * comprimento);
}
```

---

## 📦 Deploy e Build

### Opção 1: Deploy Direto (Sem Build)

**Netlify Drop:**
```bash
# Arraste os arquivos para https://app.netlify.com/drop
# Pronto! URL gerada automaticamente
```

**GitHub Pages:**
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/usuario/calculadora-furos.git
git push -u origin main

# No GitHub: Settings > Pages > Source: main branch
# URL: https://usuario.github.io/calculadora-furos/
```

**Vercel:**
```bash
npm i -g vercel
vercel
# Segue os prompts
```

### Opção 2: Build com Bundler (Se Modularizar)

**Vite (Recomendado):**
```bash
npm create vite@latest calculadora-furos -- --template vanilla
cd calculadora-furos
npm install

# Adicionar dependências
npm install three html2canvas jspdf

# Build
npm run build

# Output em dist/
```

**package.json:**
```json
{
  "name": "calculadora-furos",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "vite": "^5.0.0"
  },
  "dependencies": {
    "three": "^0.128.0",
    "html2canvas": "^1.4.1",
    "jspdf": "^2.5.1"
  }
}
```

### Opção 3: PWA para APK

**PWABuilder (Mais Fácil):**
1. Deploy o site (Netlify, Vercel, GitHub Pages)
2. Acesse https://www.pwabuilder.com/
3. Cole a URL do site
4. Clique em "Generate Package" > Android
5. Configure package ID: `com.calculadora.furos`
6. Download do APK/AAB

**Android Studio (Mais Controle):**
```java
// MainActivity.java
public class MainActivity extends AppCompatActivity {
    private WebView webView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        webView = findViewById(R.id.webview);
        WebSettings settings = webView.getSettings();
        settings.setJavaScriptEnabled(true);
        settings.setDomStorageEnabled(true);
        settings.setAllowFileAccess(true);

        webView.setWebViewClient(new WebViewClient());
        webView.loadUrl("file:///android_asset/index.html");
    }
}
```

### CI/CD com GitHub Actions

```.github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./
```

---

## 🐛 Troubleshooting

### Problemas Comuns

#### 1. **3D não renderiza no mobile**

**Causa:** WebGL não suportado ou desabilitado

**Solução:**
```javascript
function verificarWebGL() {
  const canvas = document.createElement('canvas');
  const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');

  if (!gl) {
    alert('Seu dispositivo não suporta WebGL. Use a visualização 2D.');
    document.getElementById('btn3D').style.display = 'none';
  }
}
```

#### 2. **Service Worker não atualiza**

**Causa:** Cache do navegador

**Solução:**
```javascript
// Incrementar versão no sw.js
const CACHE_NAME = 'calculadora-v2'; // Era v1

// Forçar atualização
self.addEventListener('activate', event => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.filter(name => name !== CACHE_NAME)
          .map(name => caches.delete(name))
      );
    })
  );
});
```

#### 3. **PDF não gera no iOS**

**Causa:** html2canvas limitações com Safari

**Solução:**
```javascript
// Opções específicas para iOS
const options = {
  scale: 2,
  useCORS: true,
  allowTaint: true,
  backgroundColor: '#ffffff',
  logging: false,
  windowWidth: 800,
  windowHeight: 1200
};

html2canvas(elemento, options).then(canvas => {
  // Converter para jsPDF
});
```

#### 4. **Layout quebrado em landscape mobile**

**Causa:** Media query não específica suficiente

**Solução:**
```css
@media (max-width: 768px) and (orientation: landscape) {
  #canvas3d {
    height: 280px !important; /* Forçar altura menor */
  }

  .armario {
    width: 150px !important;
  }
}
```

#### 5. **Validação não dispara**

**Causa:** Event listeners não capturando inputs dinâmicos

**Solução:**
```javascript
// Usar delegação de eventos no container pai
document.getElementById('vaosContainer').addEventListener('input', function(e) {
  if (e.target.classList.contains('vao-input')) {
    validarConfiguracao();
  }
});
```

---

## 📚 Recursos Adicionais

### Documentação de APIs Usadas

- **Three.js**: https://threejs.org/docs/
- **html2canvas**: https://html2canvas.hertzen.com/documentation
- **jsPDF**: https://github.com/parallax/jsPDF
- **Service Workers**: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
- **Web App Manifest**: https://web.dev/add-manifest/
- **Wake Lock API**: https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API

### Ferramentas de Desenvolvimento

- **Lighthouse**: Auditar PWA (Chrome DevTools)
- **PWA Builder**: https://www.pwabuilder.com/
- **Can I Use**: https://caniuse.com/ (verificar suporte de APIs)
- **WebPageTest**: https://www.webpagetest.org/ (performance)

### Comunidades

- **r/webdev**: https://reddit.com/r/webdev
- **Three.js Discourse**: https://discourse.threejs.org/
- **PWA Google Group**: https://groups.google.com/g/progressive-web-apps

---

## 🤝 Contribuindo

### Como Contribuir

1. Fork o repositório
2. Crie uma branch: `git checkout -b feature/nova-funcionalidade`
3. Commit suas mudanças: `git commit -am 'Adiciona nova funcionalidade'`
4. Push para a branch: `git push origin feature/nova-funcionalidade`
5. Abra um Pull Request

### Padrões de Código

```javascript
// ✅ BOM: Nomes descritivos
function calcularPosicaoFuro(altura, vao, espessura) {
  return altura + vao + espessura + OFFSET_PARAFUSO;
}

// ❌ RUIM: Nomes genéricos
function calc(a, b, c) {
  return a + b + c + 4;
}

// ✅ BOM: Comentários explicativos
// Offset de 4mm para encaixe perfeito do parafuso
const OFFSET_PARAFUSO = 4;

// ❌ RUIM: Comentários óbvios
// Define offset como 4
const OFFSET_PARAFUSO = 4;
```

### Checklist de PR

- [ ] Código testado manualmente
- [ ] Funciona em mobile (Chrome DevTools)
- [ ] Funciona offline
- [ ] Não quebra funcionalidades existentes
- [ ] Comentários adicionados onde necessário
- [ ] README atualizado (se aplicável)

---

## 📄 Licença

Este projeto é de código aberto e está disponível sob a licença MIT.

---

## 👨‍💻 Autor

Desenvolvido para a comunidade de marcenaria brasileira.

**Contato:** (Adicionar informações de contato, se desejar)

---

**Última atualização:** Fevereiro 2026

**Versão do Documento:** 1.0
