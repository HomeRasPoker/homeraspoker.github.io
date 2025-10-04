<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Divisão de Premiação</title>
<style>
  body {
    font-family: 'Poppins', sans-serif;
    background: linear-gradient(135deg, #f0e6d2, #b28b5a);
    margin: 0;
    padding: 20px;
    color: #333;
  }
  h1 {
    text-align: center;
    color: #2e2e2e;
  }
  .container {
    max-width: 600px;
    margin: 0 auto;
    background: #fff;
    padding: 25px;
    border-radius: 15px;
    box-shadow: 0 4px 15px rgba(0,0,0,0.2);
  }
  label {
    font-weight: bold;
  }
  input {
    width: 100%;
    padding: 10px;
    margin: 8px 0 15px;
    border-radius: 10px;
    border: 1px solid #bbb;
    font-size: 16px;
  }
  button {
    width: 100%;
    padding: 12px;
    margin-top: 10px;
    border: none;
    border-radius: 10px;
    font-size: 16px;
    cursor: pointer;
    transition: 0.3s;
  }
  #calcular {
    background-color: #4CAF50;
    color: white;
  }
  #salvarQuinto {
    background-color: #007bff;
    color: white;
  }
  #desfazerQuinto {
    background-color: #d9534f;
    color: white;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
  }
  th, td {
    border: 1px solid #ccc;
    padding: 12px;
    text-align: center;
    color: #000; /* letras pretas e bem visíveis */
  }
  th {
    background-color: #f4f4f4;
    font-weight: bold;
  }
  .resultado {
    margin-top: 15px;
    text-align: center;
    font-weight: bold;
  }
</style>
</head>
<body>
  <div class="container">
    <h1>Divisão de Premiação</h1>
    <label for="valorTotal">Valor total arrecadado (R$):</label>
    <input type="number" id="valorTotal" placeholder="Digite o valor total">
    <button id="calcular">Calcular Divisão</button>
    <button id="salvarQuinto" disabled>Salvar o Quinto</button>
    <button id="desfazerQuinto" disabled>Desfazer Quinto</button>

    <div id="resultado" class="resultado"></div>
  </div>

<script>
let premiosOriginais = {};
let quintoAdicionado = false;
let valorQuinto = 0;

document.getElementById('calcular').addEventListener('click', () => {
  const total = parseFloat(document.getElementById('valorTotal').value);
  if (isNaN(total) || total <= 0) {
    alert("Digite um valor válido.");
    return;
  }

  // Descontos iniciais
  const casa = total * 0.10;
  const porquinho = total * 0.05;
  let restante = total - casa - porquinho;

  // Distribuição base
  let p1 = restante * 0.50;
  let p2 = restante * 0.25;
  let p3 = restante * 0.15;
  let p4 = restante * 0.10;

  // Arredondamento e ajuste de sobras
  p1 = Math.round(p1 / 10) * 10;
  p2 = Math.round(p2 / 10) * 10;
  p3 = Math.round(p3 / 10) * 10;
  p4 = Math.round(p4 / 10) * 10;

  const distribuido = p1 + p2 + p3 + p4;
  const sobra = restante - distribuido;

  let porquinhoFinal = porquinho + (sobra > 0 ? sobra : 0);

  premiosOriginais = { total, casa, porquinho: porquinhoFinal, p1, p2, p3, p4 };
  quintoAdicionado = false;
  valorQuinto = 0;

  mostrarTabela(premiosOriginais);
  document.getElementById('salvarQuinto').disabled = false;
  document.getElementById('desfazerQuinto').disabled = true;
});

document.getElementById('salvarQuinto').addEventListener('click', () => {
  if (quintoAdicionado) {
    alert("O quinto já foi salvo.");
    return;
  }

  let { total, casa, porquinho, p1, p2, p3, p4 } = premiosOriginais;
  const salvarValor = total < 1000 ? 40 : 100; // valor total a ser redistribuído

  // Divisão fixa (70 e 30)
  const tirarP1 = 70;
  const tirarP2 = 30;
  const quinto = tirarP1 + tirarP2;

  // Garantir que não quebre a regra
  const novoP1 = p1 - tirarP1;
  const novoP2 = p2 - tirarP2;
  const novoP3 = p3;
  const novoP4 = p4;

  if (quinto >= novoP4) {
    alert("Não é possível salvar o quinto — valor igual ou maior que o quarto lugar.");
    return;
  }

  // Mantém o total distribuído igual
  const distribuidoAntigo = p1 + p2 + p3 + p4;
  const distribuidoNovo = novoP1 + novoP2 + novoP3 + novoP4 + quinto;
  const ajuste = distribuidoAntigo - distribuidoNovo;
  porquinho += ajuste;

  premiosOriginais = { total, casa, porquinho, p1: novoP1, p2: novoP2, p3: novoP3, p4: novoP4, p5: quinto };
  quintoAdicionado = true;
  valorQuinto = quinto;

  mostrarTabela(premiosOriginais);
  document.getElementById('salvarQuinto').disabled = true;
  document.getElementById('desfazerQuinto').disabled = false;
});

document.getElementById('desfazerQuinto').addEventListener('click', () => {
  if (!quintoAdicionado) {
    alert("Nenhum quinto foi salvo ainda.");
    return;
  }

  const { p1, p2, p3, p4, total, casa, porquinho } = premiosOriginais;
  // Restaura para valores originais (sem quinto)
  document.getElementById('calcular').click();
  document.getElementById('salvarQuinto').disabled = false;
  document.getElementById('desfazerQuinto').disabled = true;
});

function mostrarTabela({ total, casa, porquinho, p1, p2, p3, p4, p5 }) {
  let html = `
    <table>
      <tr><th>Posição</th><th>Prêmio (R$)</th></tr>
      <tr><td>1º Lugar</td><td>${p1?.toFixed(2) || '-'}</td></tr>
      <tr><td>2º Lugar</td><td>${p2?.toFixed(2) || '-'}</td></tr>
      <tr><td>3º Lugar</td><td>${p3?.toFixed(2) || '-'}</td></tr>
      <tr><td>4º Lugar</td><td>${p4?.toFixed(2) || '-'}</td></tr>
      ${p5 ? `<tr><td>5º Lugar</td><td>${p5.toFixed(2)}</td></tr>` : ''}
    </table>
    <p>Casa: R$ ${casa.toFixed(2)} | Porquinho: R$ ${porquinho.toFixed(2)} | Total: R$ ${total.toFixed(2)}</p>
  `;
  document.getElementById('resultado').innerHTML = html;
}
</script>
</body>
</html>
