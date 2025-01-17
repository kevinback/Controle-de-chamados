<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Chamados</title>
    <style>
        /* Estilos gerais */
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            height: 100vh;
            background-color: #f4f4f4;
            display: flex;
            justify-content: center; /* Centraliza horizontalmente */
            align-items: flex-start; /* Alinha o conteúdo no topo */
            flex-direction: column;
            padding-top: 20px; /* Espaçamento extra no topo */
            overflow: auto;
        }

        .container {
            display: flex;
            width: 90%;
            max-width: 1200px;
            justify-content: space-between;
            margin-bottom: 30px;
        }

        .form-container, .list-container {
            flex: 1;
            padding: 20px;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        .form-container {
            margin-right: 20px;
        }

        h1, h2, h3 {
            text-align: center;
            color: #333;
            margin-bottom: 20px;
        }

        form {
            display: flex;
            flex-direction: column;
        }

        label {
            margin: 10px 0 5px;
            color: #333;
        }

        input, select, button, textarea {
            padding: 10px;
            margin: 5px 0 15px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        button {
            background-color: #4CAF50;
            color: white;
            cursor: pointer;
        }

        button:hover {
            background-color: #45a049;
        }

        #usuariosCadastrados {
            margin-top: 20px;
            padding: 10px;
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 5px;
            max-height: 500px;
            overflow-y: auto;
        }

        .delete-btn {
            color: red;
            font-size: 16px;
            cursor: pointer;
            background: none;
            border: none;
            padding: 0;
        }

        .pagination {
            display: flex;
            justify-content: center;
            margin-top: 20px;
        }

        .pagination button {
            padding: 10px;
            margin: 0 5px;
            border: 1px solid #ccc;
            border-radius: 4px;
            cursor: pointer;
        }

        .pagination button:hover {
            background-color: #ddd;
        }

        .container .list-container {
            margin-top: 50px;
        }

    </style>
</head>
<body>
    <div class="container">
        <div class="form-container">
            <h1>Controle de Chamados</h1>
            <h2>Cadastro de Chamados</h2>
            <form id="cadastroForm">
                <label for="numeroChamado">Número do Chamado:</label>
                <input type="text" id="numeroChamado" name="numeroChamado" required>

                <label for="dataChamado">Data do Chamado:</label>
                <input type="date" id="dataChamado" name="dataChamado" required>

                <label for="nomeTecnico">Nome do Técnico:</label>
                <input type="text" id="nomeTecnico" name="nomeTecnico" required>

                <label for="endereco">Endereço:</label>
                <input type="text" id="endereco" name="endereco" required>

                <label for="km">KM:</label>
                <input type="number" id="km" name="km" required>

                <label for="descricaoChamado">Descrição do Chamado:</label>
                <textarea id="descricaoChamado" name="descricaoChamado" rows="4" required></textarea>

                <button type="submit">Cadastrar</button>
            </form>
        </div>

        <div class="list-container">
            <h3>Chamados Cadastrados</h3>
            <div id="usuariosCadastrados"></div>

            <div class="pagination">
                <button id="prevPage" onclick="mudarPagina('prev')" disabled>Anterior</button>
                <button id="nextPage" onclick="mudarPagina('next')">Próxima</button>
            </div>
        </div>
    </div>

    <script>
        let paginaAtual = 1;
        const CHAMADOS_POR_PAGINA = 10;

        // Função para salvar os dados no localStorage
        function salvarCadastro(numeroChamado, dataChamado, nomeTecnico, endereco, km, descricaoChamado) {
            let chamados = JSON.parse(localStorage.getItem("chamados")) || [];
            const chamado = { numeroChamado, dataChamado, nomeTecnico, endereco, km, descricaoChamado };
            chamados.push(chamado);
            localStorage.setItem("chamados", JSON.stringify(chamados));
        }

        // Função para excluir um chamado do localStorage
        function excluirChamado(index) {
            if (confirm("Deseja mesmo excluir este chamado?")) {
                let chamados = JSON.parse(localStorage.getItem("chamados")) || [];
                chamados.splice(index, 1); // Remove o chamado com o índice fornecido
                localStorage.setItem("chamados", JSON.stringify(chamados));
                exibirChamados(); // Atualiza a lista de chamados
            }
        }

        // Função para exibir os chamados cadastrados com paginação
        function exibirChamados() {
            const chamados = JSON.parse(localStorage.getItem("chamados")) || [];
            const divChamados = document.getElementById("usuariosCadastrados");
            divChamados.innerHTML = ""; // Limpa a área antes de exibir

            // Calcula o índice inicial e final da página atual
            const inicio = (paginaAtual - 1) * CHAMADOS_POR_PAGINA;
            const fim = inicio + CHAMADOS_POR_PAGINA;
            const chamadosPaginaAtual = chamados.slice(inicio, fim);

            if (chamadosPaginaAtual.length === 0) {
                divChamados.innerHTML = "<p>Nenhum chamado cadastrado.</p>";
            } else {
                chamadosPaginaAtual.forEach((chamado, index) => {
                    divChamados.innerHTML += `
                        <p><strong>Chamado #${chamado.numeroChamado}</strong><br>
                        Data do Chamado: ${chamado.dataChamado}<br>
                        Nome do Técnico: ${chamado.nomeTecnico}<br>
                        Endereço: ${chamado.endereco}<br>
                        KM: ${chamado.km}<br>
                        Descrição do Chamado: ${chamado.descricaoChamado}
                        <button class="delete-btn" onclick="excluirChamado(${index + inicio})">X</button></p>
                        <hr>
                    `;
                });
            }

            // Controle de habilitação/desabilitação dos botões de navegação
            document.getElementById("prevPage").disabled = paginaAtual === 1;
            document.getElementById("nextPage").disabled = paginaAtual * CHAMADOS_POR_PAGINA >= chamados.length;
        }

        // Função para navegar entre as páginas
        function mudarPagina(direcao) {
            const chamados = JSON.parse(localStorage.getItem("chamados")) || [];
            if (direcao === 'next' && paginaAtual * CHAMADOS_POR_PAGINA < chamados.length) {
                paginaAtual++;
            } else if (direcao === 'prev' && paginaAtual > 1) {
                paginaAtual--;
            }
            exibirChamados();
        }

        // Função para lidar com o envio do formulário
        document.getElementById("cadastroForm").addEventListener("submit", function (event) {
            event.preventDefault();

            const numeroChamado = document.getElementById("numeroChamado").value;
            const dataChamado = document.getElementById("dataChamado").value;
            const nomeTecnico = document.getElementById("nomeTecnico").value;
            const endereco = document.getElementById("endereco").value;
            const km = document.getElementById("km").value;
            const descricaoChamado = document.getElementById("descricaoChamado").value;

            if (numeroChamado && dataChamado && nomeTecnico && endereco && km && descricaoChamado) {
                salvarCadastro(numeroChamado, dataChamado, nomeTecnico, endereco, km, descricaoChamado);
                document.getElementById("cadastroForm").reset();
                exibirChamados();
            } else {
                alert("Por favor, preencha todos os campos.");
            }
        });

        // Chama a função para exibir os chamados cadastrados ao carregar a página
        window.onload = exibirChamados;
    </script>
</body>
</html>
