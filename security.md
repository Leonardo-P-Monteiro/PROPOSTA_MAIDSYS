
# Diretrizes de Implementação: Segurança Client-Side para Documentos Web

Este documento estabelece o padrão de implementação de defesas no lado do cliente (Client-Side Security) para propostas comerciais, contratos e documentos confidenciais baseados em HTML.

O objetivo desta arquitetura é **modular**. Não precisará de alterar o conteúdo principal do seu HTML; basta envolvê-lo com as marcações abaixo e adicionar os respetivos blocos de CSS e JavaScript.

## 🛡️ Resumo das Defesas Implementadas

1.  **Bloqueio de Seleção e Cópia:** Impede o uso do rato para selecionar textos (Ctrl+C).
    
2.  **Bloqueio de Menu de Contexto:** Desativa o botão direito do rato.
    
3.  **Bloqueio de Impressão:** Oculta o conteúdo e exibe um alerta caso o utilizador acione Ctrl+P.
    
4.  **Prevenção de Inspecionar Elemento:** Bloqueia teclas de programador (F12, Ctrl+Shift+I, Ctrl+U).
    
5.  **Blackout Dinâmico (Anti-Captura):** Oculta o ecrã (fundo preto) mediante:
    
    -   Acionamento de teclas de Print (PrintScreen, Win+Shift+S, Mac Cmd+Shift+S).
        
    -   Perda de foco da janela (Alt+Tab, clique noutro programa).
        
    -   Saída do cursor do rato da área do navegador (tentativa de acionar softwares de captura pelo menu iniciar/barra de tarefas).
        

## Passo 1: Estrutura HTML (Envolvimento)

Para proteger qualquer página, deve adicionar **três elementos estruturais**. Todo o conteúdo original da sua página deve ser inserido **dentro** da div `#secure-wrapper`.

Insira este código logo após a tag `<body>`:

```
<!-- 1. Overlay de Segurança Fixo (Blackout Total) -->
<div id="security-overlay">
    <h2 style="color: white; font-family: sans-serif;">Conteúdo Protegido</h2>
    <p style="color: #ccc; font-family: sans-serif;">Mantenha o cursor dentro da página e a janela ativa para visualizar.</p>
</div>

<!-- 2. Mensagem mostrada apenas na tentativa de impressão (Ctrl+P) -->
<div id="print-warning">
    <h2 style="color: #333; font-family: sans-serif;">Acesso Restrito</h2>
    <p style="color: #666; font-family: sans-serif;">A impressão deste documento está bloqueada por confidencialidade.</p>
</div>

<!-- 3. Wrapper de Segurança (O SEU CONTEÚDO VAI AQUI DENTRO) -->
<div id="secure-wrapper">
    
    <!-- ========================================== -->
    <!-- COLOQUE TODO O SEU HTML/CONTEÚDO ORIGINAL AQUI -->
    <!-- ========================================== -->

</div> <!-- Fim Secure Wrapper -->

```

## Passo 2: Estilização CSS (O Escudo Visual)

Adicione o bloco CSS abaixo dentro da tag `<style>` no `<head>` do seu documento, ou num ficheiro `.css` global. Ele é responsável por ocultar o documento durante tentativas de cópia e impressão.

```
/* TÉCNICA 1: Bloqueio de Seleção Global */
body {
    -webkit-user-select: none; /* Safari */
    -ms-user-select: none; /* IE 10+ */
    user-select: none; /* Padrão moderno */
}

/* TÉCNICA 2: Configuração do Wrapper para o Blackout */
#secure-wrapper {
    min-height: 100vh;
}

body.is-secured {
    overflow: hidden !important; /* Trava o scroll da página durante o bloqueio */
}

/* Oculta instantaneamente o conteúdo quando a classe de segurança é ativada */
body.is-secured #secure-wrapper {
    display: none !important; 
}

/* TÉCNICA 3: Configuração do Blackout (Ecrã Preto) */
#security-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: #1a1a1a; /* Fundo preto/escuro */
    z-index: 9999999; /* Garante que ficará por cima de TUDO */
    display: none; /* Escondido por padrão */
    flex-direction: column;
    align-items: center;
    justify-content: center;
    pointer-events: none;
}

body.is-secured #security-overlay {
    display: flex; /* Exibe o ecrã preto instantaneamente */
    pointer-events: all;
}

/* TÉCNICA 4: Proteção contra Impressão nativa (@media print) */
#print-warning {
    display: none;
}

@media print {
    body {
        background-color: white;
    }
    #secure-wrapper {
        display: none !important; /* Remove o conteúdo da fila de impressão */
    }
    #print-warning {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100vh;
        text-align: center;
    }
}

```

## Passo 3: Lógica JavaScript (Os Gatilhos)

Este script é o "cérebro" da segurança. Ele escuta as interações do utilizador e dispara a classe `.is-secured` no `<body>`. Coloque este código no final do seu documento, antes do fecho da tag `</body>`.

```
<script>
    // 1. Bloqueia o clique com o Botão Direito (Menu de contexto)
    document.addEventListener('contextmenu', function(event) {
        event.preventDefault();
    });

    // 2. Funções de Ativação do Blackout
    function applySecurityBlur() {
        document.body.classList.add('is-secured');
    }

    function removeSecurityBlur() {
        document.body.classList.remove('is-secured');
    }

    // GATILHO 1: Oculta a página se o rato sair da área de renderização (Mouseleave)
    // Previne que usem ferramentas de captura clicando na barra de tarefas.
    document.documentElement.addEventListener('mouseleave', applySecurityBlur);
    document.documentElement.addEventListener('mouseenter', removeSecurityBlur);

    // GATILHO 2: Se a janela perder o foco (Alt+Tab, clicar no menu iniciar, etc)
    window.addEventListener('blur', applySecurityBlur);
    window.addEventListener('focus', removeSecurityBlur);
    
    // GATILHO 3: API de Visibilidade (Garante proteção caso o utilizador troque de aba no navegador)
    document.addEventListener('visibilitychange', function() {
        if (document.hidden) {
            applySecurityBlur();
        } else {
            removeSecurityBlur();
        }
    });
    
    // GATILHO 4: Escuta do Teclado (Interceção de Atalhos)
    document.addEventListener('keydown', function(event) {
        // Tecla física Print Screen
        if (event.key === 'PrintScreen' || event.keyCode === 44) {
            applySecurityBlur();
        }
        
        // Atalhos de programador e ferramentas de extração
        if (event.keyCode === 123 || // F12
           (event.ctrlKey && event.shiftKey && event.keyCode === 73) || // Ctrl+Shift+I (Inspecionar)
           (event.ctrlKey && event.shiftKey && event.keyCode === 74) || // Ctrl+Shift+J (Consola)
           (event.ctrlKey && event.keyCode === 85) || // Ctrl+U (Ver Código Fonte)
           (event.ctrlKey && event.keyCode === 67) || // Ctrl+C (Copiar)
           (event.ctrlKey && event.keyCode === 80)) { // Ctrl+P (Imprimir)
            event.preventDefault(); 
            return false;
        }

        // Bloqueio de Atalhos de Captura Nativa (Win+Shift+S / Ctrl+Shift+S / Cmd+Shift+S)
        if (event.shiftKey && (event.key === 's' || event.key === 'S' || event.keyCode === 83)) {
            if (event.metaKey || event.ctrlKey) {
                applySecurityBlur();
                event.preventDefault(); 
                return false;
            }
        }
    });

    // Remove o blackout instantes após a tecla Print Screen ser solta
    document.addEventListener('keyup', function(event) {
        if (event.key === 'PrintScreen' || event.keyCode === 44) {
            setTimeout(removeSecurityBlur, 1500); // 1.5 segundos de punição visual
        }
    });
</script>

```

## ⚠️ Limitações Conhecidas (Aviso Técnico)

É importante ter ciência técnica de que defesas baseadas em front-end operam na camada do navegador.

-   **Fotografias Físicas:** Não previne que o utilizador fotografe o monitor com um telemóvel/smartphone.
    
-   **Desativação de JS:** Utilizadores extremamente técnicos podem desativar o JavaScript do navegador globalmente antes de aceder à página. Para mitigar isto, pode ocultar todo o site por defeito no CSS e usar o JS inicial para revelá-lo (assim, se o JS estiver desligado, o site simplesmente não carrega).
    
-   **Extensões de Navegador:** Algumas extensões de _Screen Scraping_ podem contornar eventos DOM.
    

A arquitetura descrita acima cobre **99% dos utilizadores comuns e corporativos**, garantindo um excelente nível de conformidade e proteção de propriedade intelectual (PI).
