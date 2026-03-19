
# Prompt de Instrução: Implementação de Segurança Client-Side para Documentos Web

**Contexto para a IA:** Aja como um Engenheiro de Segurança Front-end. A sua tarefa é receber um código HTML (uma proposta comercial, contrato ou documento confidencial) e implementar uma arquitetura de defesas no lado do cliente (Client-Side Security) para evitar cópias, capturas de ecrã e impressões.

**Regra de Ouro:** A arquitetura é estritamente **modular**. Você NÃO deve alterar o conteúdo principal, o design ou a lógica de negócio do HTML fornecido pelo utilizador. A sua tarefa é apenas envolver o conteúdo existente com as marcações abaixo e injetar os respetivos blocos de CSS e JavaScript.

## 🛡️ Resumo das Defesas a Implementar

Ao processar o código fornecido, certifique-se de que a página final cumpra os seguintes requisitos de segurança:

1.  **Bloqueio de Seleção e Cópia:** Impedir o uso do rato para selecionar textos (Ctrl+C).
    
2.  **Bloqueio de Menu de Contexto:** Desativar o botão direito do rato.
    
3.  **Bloqueio de Impressão:** Ocultar o conteúdo e exibir um alerta caso o utilizador acione Ctrl+P.
    
4.  **Prevenção de Inspecionar Elemento:** Bloquear teclas de programador (F12, Ctrl+Shift+I, Ctrl+U).
    
5.  **Blackout Dinâmico (Anti-Captura):** Ocultar o ecrã (aplicar fundo preto) mediante:
    
    -   Perda de foco da janela (Alt+Tab, clique noutro programa).
        
    -   Saída do cursor do rato da área do navegador (tentativa de acionar softwares de captura pelo menu iniciar/barra de tarefas).
        
    -   Acionamento de teclas de Print (PrintScreen).
        
6.  **Penalidade de Captura (NOVO):** Se for detetada uma combinação de teclas de captura nativa do sistema operativo (Win+Shift ou Meta+Shift), o sistema aplica um "castigo" de 10 minutos. Durante este período, a tela permanecerá ofuscada com um aviso de violação orientando a recarregar a página, ignorando movimentos do rato.
    

Siga os três passos abaixo para modificar o código HTML fornecido:

## Passo 1: Estrutura HTML (Envolvimento)

Injete **três elementos estruturais** logo após a abertura da tag `<body>`.

Todo o conteúdo original da página do utilizador deve ser movido para **dentro** da div `#secure-wrapper`. Note que os IDs `security-title` e `security-message` são obrigatórios para a injeção do aviso de penalidade.

```
<!-- 1. Overlay de Segurança Fixo (Blackout Total) -->
<div id="security-overlay">
    <h2 id="security-title" style="color: white; font-family: sans-serif;">Conteúdo Protegido</h2>
    <p id="security-message" style="color: #ccc; font-family: sans-serif;">Mantenha o cursor dentro da página e a janela ativa para visualizar.</p>
</div>

<!-- 2. Mensagem mostrada apenas na tentativa de impressão (Ctrl+P) -->
<div id="print-warning">
    <h2 style="color: #333; font-family: sans-serif;">Acesso Restrito</h2>
    <p style="color: #666; font-family: sans-serif;">A impressão deste documento está bloqueada por confidencialidade.</p>
</div>

<!-- 3. Wrapper de Segurança (O CONTEÚDO ORIGINAL DEVE FICAR AQUI DENTRO) -->
<div id="secure-wrapper">
    
    <!-- ========================================== -->
    <!-- CONTEÚDO ORIGINAL DO UTILIZADOR AQUI       -->
    <!-- ========================================== -->

</div> <!-- Fim Secure Wrapper -->

```

## Passo 2: Estilização CSS (O Escudo Visual)

Adicione o bloco CSS abaixo dentro da tag `<style>` no `<head>` do documento. Este CSS será responsável por ocultar o documento durante tentativas de cópia e impressão, além de remover a funcionalidade de seleção nativa.

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
    text-align: center;
    padding: 20px;
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

## Passo 3: Lógica JavaScript (Os Gatilhos e Timer de Penalidade)

Injete o script abaixo no final do documento fornecido, imediatamente antes do fecho da tag `</body>`. Ele atuará como o "cérebro" da segurança e fará a gestão do bloqueio severo (10 minutos) caso identifique tentativas de captura.

```
<script>
    // Variáveis de Controle de Penalidade
    let isPenaltyActive = false;
    let penaltyTimer = null;

    // 1. Bloqueia o clique com o Botão Direito (Menu de contexto)
    document.addEventListener('contextmenu', function(event) {
        event.preventDefault();
    });

    // 2. Funções de Ativação do Blackout
    function applySecurityBlur() {
        document.body.classList.add('is-secured');
    }

    function removeSecurityBlur() {
        // Impede o desbloqueio manual caso a penalidade de 10 minutos esteja ativa
        if (isPenaltyActive) return; 
        document.body.classList.remove('is-secured');
    }

    // 3. Função de Penalidade (10 Minutos)
    function triggerPenalty() {
        isPenaltyActive = true;
        applySecurityBlur();
        
        // Altera as mensagens do overlay dinamicamente
        const titleEl = document.getElementById('security-title');
        const msgEl = document.getElementById('security-message');
        const originalTitle = "Conteúdo Protegido";
        const originalMsg = "Mantenha o cursor dentro da página e a janela ativa para visualizar.";
        
        if(titleEl && msgEl) {
            titleEl.innerText = "Tentativa de Captura Detetada";
            msgEl.innerHTML = "O conteúdo foi bloqueado por segurança.<br>Ele será liberado automaticamente em 10 minutos.";
            msgEl.style.color = "#fa5a1d"; // Cor de alerta
        }

        clearTimeout(penaltyTimer);
        
        // Timer de 10 minutos (600.000 milissegundos)
        penaltyTimer = setTimeout(function() {
            isPenaltyActive = false;
            
            // Restaura os textos originais
            if(titleEl && msgEl) {
                titleEl.innerText = originalTitle;
                msgEl.innerText = originalMsg;
                msgEl.style.color = "#ccc";
            }
            
            // Remove o blackout caso o utilizador esteja ativamente na tela
            if (!document.hidden && document.hasFocus()) {
                removeSecurityBlur();
            }
        }, 600000); 
    }

    // GATILHO 1: Oculta a página se o rato sair da área de renderização
    document.documentElement.addEventListener('mouseleave', applySecurityBlur);
    document.documentElement.addEventListener('mouseenter', removeSecurityBlur);

    // GATILHO 2: Se a janela perder o foco (Alt+Tab)
    window.addEventListener('blur', applySecurityBlur);
    window.addEventListener('focus', removeSecurityBlur);
    
    // GATILHO 3: API de Visibilidade (Troca de aba)
    document.addEventListener('visibilitychange', function() {
        if (document.hidden) {
            applySecurityBlur();
        } else {
            removeSecurityBlur();
        }
    });
    
    // GATILHO 4: Escuta do Teclado (Interceção de Atalhos e Capturas)
    document.addEventListener('keydown', function(event) {
        // REGRA DE PENALIDADE: Win+Shift ou Cmd+Shift (Ex: Win+Shift+S)
        if (event.metaKey && event.shiftKey) {
            triggerPenalty();
            event.preventDefault();
            return false;
        }

        // Tecla física Print Screen
        if (event.key === 'PrintScreen' || event.keyCode === 44) {
            applySecurityBlur();
        }
        
        // Atalhos de programador e ferramentas de extração
        if (event.keyCode === 123 || // F12
           (event.ctrlKey && event.shiftKey && event.keyCode === 73) || // Ctrl+Shift+I
           (event.ctrlKey && event.shiftKey && event.keyCode === 74) || // Ctrl+Shift+J
           (event.ctrlKey && event.keyCode === 85) || // Ctrl+U
           (event.ctrlKey && event.keyCode === 67) || // Ctrl+C
           (event.ctrlKey && event.keyCode === 80)) { // Ctrl+P
            event.preventDefault(); 
            return false;
        }

        // Bloqueio Ctrl+Shift+S (Redundância para outros SOs/Browsers)
        if (event.shiftKey && (event.key === 's' || event.key === 'S' || event.keyCode === 83)) {
            if (event.metaKey || event.ctrlKey) {
                applySecurityBlur();
                event.preventDefault(); 
                return false;
            }
        }
    });

    // Liberta a tecla Print Screen apenas se não houver penalidade a decorrer
    document.addEventListener('keyup', function(event) {
        if (event.key === 'PrintScreen' || event.keyCode === 44) {
            setTimeout(removeSecurityBlur, 1500);
        }
    });
</script>

```

## Diretrizes Finais de Revisão da IA:

1.  Certifique-se de não destruir estilos CSS originais (apenas anexe os novos estilos).
    
2.  Verifique se as dependências do documento original (Bootstrap, Tailwind, jQuery, etc.) continuam a funcionar.
    
3.  **Atenção ao DOM:** Se o utilizador fornecer um projeto complexo com scripts em linha ou modais, garanta que a div `#secure-wrapper` encerra apenas imediatamente antes da injeção das tags `<script>` do rodapé, para não quebrar scripts que dependam da hierarquia do corpo do documento.
