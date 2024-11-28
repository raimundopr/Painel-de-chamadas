# Painel-de-chamadas
PAINEL DE CHAMADAS
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel de Chamada</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.4.0/jspdf.umd.min.js"></script>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(to right, #4facfe, #00f2fe);
            color: #333;
            margin: 0;
            padding: 0;
        }
        .container {
            max-width: 900px;
            margin: 30px auto;
            padding: 20px;
            background: #fff;
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
        }
        h1 {
            text-align: center;
            color: #007BFF;
        }
        .buttons {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-around;
            margin: 20px 0;
        }
        .btn {
            padding: 10px 20px;
            font-size: 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            color: #fff;
            margin: 5px;
        }
        .btn-normal {
            background-color: #28a745;
        }
        .btn-preferencial {
            background-color: #dc3545;
        }
        .btn-secondary {
            background-color: #007bff;
        }
        .btn:hover {
            opacity: 0.9;
        }
        .mesa-selection {
            display: flex;
            justify-content: center;
            align-items: center;
            margin: 15px 0;
        }
        .mesa-selection label {
            margin-right: 10px;
        }
        select {
            padding: 5px 10px;
            font-size: 16px;
        }
        .ficha-display {
            text-align: center;
            margin-top: 20px;
            font-size: 20px;
        }
        .ficha-display span {
            display: block;
            font-size: 24px;
            margin: 10px 0;
        }
        #fichaAtual {
            color: #dc3545;
            font-weight: bold;
        }
        #fichaAnterior {
            color: #28a745;
            font-weight: bold;
        }
        .relatorio {
            margin-top: 30px;
        }
        .relatorio-title {
            font-size: 20px;
            font-weight: bold;
            margin-bottom: 10px;
        }
        select#listaFichas {
            width: 100%;
            margin-top: 10px;
            cursor: pointer;
        }
        .ficha-options {
            margin: 10px;
        }
        .ficha-options select {
            width: 100%;
            padding: 5px;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>COLÔNIA DE PESCADORES Z 57</h1>

        <div class="mesa-selection">
            <label for="mesa">Escolha a Mesa:</label>
            <select id="mesa">
                <option value="Mesa 1">Mesa 1</option>
                <option value="Mesa 2">Mesa 2</option>
            </select>
        </div>

        <div class="buttons">
            <button class="btn btn-normal" onclick="gerarFicha('normal')">Gerar Ficha Normal</button>
            <button class="btn btn-preferencial" onclick="gerarFicha('preferencial')">Gerar Ficha Preferencial</button>
            <button class="btn btn-secondary" onclick="chamarFicha()">Chamar Ficha</button>
            <button class="btn btn-secondary" onclick="rechamarFicha()">Rechamar Ficha</button>
            <button class="btn btn-secondary" onclick="excluirFicha()">Excluir Ficha</button>
            <button class="btn btn-secondary" onclick="imprimirFichas()">Imprimir Fichas Geradas</button>
            <button class="btn btn-secondary" onclick="imprimirFichaIndividual()">Imprimir Ficha Individual</button>
            <button class="btn btn-secondary" onclick="zerarContador()">Zerar Contador</button>

        </div>

        <div class="ficha-display">
            <span><strong>Ficha Atual:</strong> <span id="fichaAtual">Nenhuma</span></span>
            <span><strong>Ficha Anterior:</strong> <span id="fichaAnterior">Nenhuma</span></span>
        </div>

        <div class="relatorio">
            <div class="relatorio-title">Fichas Geradas</div>
            <select id="listaFichas" size="10" onchange="selecionarFicha(event)">
                <!-- As fichas serão adicionadas aqui dinamicamente -->
            </select>
        </div>
    </div>

    <audio id="ding" src="https://www.soundjay.com/button/beep-07.mp3"></audio>

    <script>
        let numeroNormal = JSON.parse(localStorage.getItem('numeroNormal')) || 0;
        let numeroPreferencial = JSON.parse(localStorage.getItem('numeroPreferencial')) || 0;
        let fila = JSON.parse(localStorage.getItem('fila')) || [];
        let ultimaFichaChamada = null;
        let fichaSelecionadaParaImprimir = null;

        function salvarDados() {
            localStorage.setItem('numeroNormal', JSON.stringify(numeroNormal));
            localStorage.setItem('numeroPreferencial', JSON.stringify(numeroPreferencial));
            localStorage.setItem('fila', JSON.stringify(fila));
        }

        function gerarFicha(tipo) {
            const mesaSelecionada = document.getElementById('mesa').value;

            if (!mesaSelecionada) {
                alert('Por favor, selecione uma mesa antes de gerar a ficha.');
                return;
            }

            let numero;
            if (tipo === 'normal') {
                numeroNormal++;
                numero = `N${numeroNormal.toString().padStart(3, '0')}`;
            } else {
                numeroPreferencial++;
                numero = `P${numeroPreferencial.toString().padStart(3, '0')}`;
            }

            const ficha = {
                tipo,
                numero,
                mesa: mesaSelecionada,
                dataHora: new Date().toLocaleString('pt-BR'),
            };

            fila.push(ficha);
            salvarDados();
            atualizarRelatorio();
        }

        function chamarFicha() {
            if (fila.length === 0) {
                alert('Não há fichas na fila.');
                return;
            }

            fila.sort((a, b) => (a.tipo === 'preferencial' && b.tipo === 'normal' ? -1 : 1));

            const ficha = fila.shift();
            ultimaFichaChamada = ficha;

            const ding = document.getElementById('ding');
            ding.play();

            setTimeout(() => {
                document.getElementById('fichaAnterior').textContent = document.getElementById('fichaAtual').textContent;
                document.getElementById('fichaAtual').textContent = `${ficha.numero} (${ficha.mesa})`;
                salvarDados();
                atualizarRelatorio();
                narrar(`Ficha ${ficha.numero}, ${ficha.mesa}`);
            }, 1000);
        }

        function rechamarFicha() {
            if (!ultimaFichaChamada) {
                alert('Nenhuma ficha foi chamada ainda.');
                return;
            }
            narrar(`Rechamando ficha ${ultimaFichaChamada.numero}, ${ultimaFichaChamada.mesa}`);
        }

        function excluirFicha() {
            const lista = document.getElementById('listaFichas');
            const fichaSelecionada = lista.value;

            if (!fichaSelecionada) {
                alert('Por favor, selecione uma ficha para excluir.');
                return;
            }

            fila = fila.filter(ficha => ficha.numero !== fichaSelecionada);
            salvarDados();
            atualizarRelatorio();
        }

       function imprimirFichas() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    let linha = 20;

    doc.setFontSize(18);
    doc.text("COLÔNIA DE PESCADORES Z 57", 105, linha, { align: "center" });

    fila.forEach((ficha, index) => {
        if (linha + 40 > 280) { // Quando próximo ao final da página, cria uma nova
            doc.addPage();
            linha = 20;
            doc.text("COLÔNIA DE PESCADORES Z 57", 105, linha, { align: "center" });
        }

        linha += 20;
        doc.setFontSize(12);
        doc.text("-------------------------------------------------", 20, linha);
        linha += 7;
        doc.text("COLÔNIA DE PESCADORES Z 57", 20, linha);
        linha += 10;
        doc.text(`FICHA ${ficha.numero} ${ficha.mesa}`, 20, linha);
        linha += 10;
        doc.text(`Data e Hora: ${ficha.dataHora}`, 20, linha);
        linha += 10;
        doc.text("Priorize o silêncio e aguarde sua vez", 20, linha);
        linha += 10;
        doc.text("-------------------------------------------------", 20, linha);
        linha += 7;
    });

    doc.save("relatorio_fichas.pdf");
}

function zerarContador() {
    if (confirm("Deseja realmente zerar o contador?")) {
        numeroNormal = 0;
        numeroPreferencial = 0;
        fila = [];
        salvarDados();
        atualizarRelatorio();
        alert("Contadores zerados com sucesso!");
    }
}


        function imprimirFichaIndividual() {
            if (!fichaSelecionadaParaImprimir) {
                alert('Por favor, selecione uma ficha para imprimir.');
                return;
            }

            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();

            doc.setFontSize(18);
            doc.text(`Ficha Individual: ${fichaSelecionadaParaImprimir.numero}`, 20, 20);
            doc.setFontSize(12);
            doc.text(`Tipo: ${fichaSelecionadaParaImprimir.tipo}`, 20, 30);
            doc.text(`Mesa: ${fichaSelecionadaParaImprimir.mesa}`, 20, 40);
            doc.text(`Data e Hora: ${fichaSelecionadaParaImprimir.dataHora}`, 20, 50);

            doc.save(`ficha_${fichaSelecionadaParaImprimir.numero}.pdf`);
        }

        function atualizarRelatorio() {
            const listaFichas = document.getElementById('listaFichas');
            listaFichas.innerHTML = '';

            fila.forEach(ficha => {
                const option = document.createElement('option');
                option.textContent = ficha.numero;
                option.value = ficha.numero;
                listaFichas.appendChild(option);
            });
        }

        function selecionarFicha(event) {
            const fichaSelecionada = event.target.value;
            if (!fichaSelecionada) return;

            fichaSelecionadaParaImprimir = fila.find(f => f.numero === fichaSelecionada);
            document.getElementById('fichaAtual').textContent = `${fichaSelecionadaParaImprimir.numero} (${fichaSelecionadaParaImprimir.mesa})`;
        }

        function narrar(texto) {
            const speech = new SpeechSynthesisUtterance(texto);
            speech.lang = 'pt-BR';
            window.speechSynthesis.speak(speech);
        }

        // Inicializa o painel ao carregar
        atualizarRelatorio();
    </script>
</body>
</html>
