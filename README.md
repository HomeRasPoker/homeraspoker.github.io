<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Divisão de Premiação</title>
<style>
  body {
    font-family: 'Poppins', sans-serif;
    background: url('https://i.imgur.com/SGbM7aF.jpg') no-repeat center center fixed;
    background-size: cover;
    margin: 0;
    padding: 20px;
    color: #fff;
  }

  .container {
    max-width: 600px;
    margin: auto;
    background: rgba(60, 40, 20, 0.9);
    border-radius: 15px;
    box-shadow: 0 0 20px rgba(0,0,0,0.5);
    padding: 25px;
    text-align: center;
  }

  h1 {
    color: #d4af37;
    text-shadow: 2px 2px 4px #000;
  }

  label {
    font-weight: bold;
    font-size: 16px;
    color: #f8f8f8;
  }

  input {
    width: 100%;
    padding: 10px;
    margin: 10px 0 20px;
    border-radius: 8px;
    border: 1px solid #d4af37;
    font-size: 16px;
    text-align: center;
  }

  button {
    width: 100%;
    padding: 12px;
    border: none;
    border-radius: 10px;
    font-size: 16px;
    font-weight: bold;
    color: #fff;
    cursor: pointer;
    transition: 0.3s;
    margin-top: 10px;
  }

  #calcular { background-color: #388e3c; }
  #salvarQuinto { background-color: #c49102; }
  #desfazerQuinto { background-color: #a93226; }

  button:hover {
    opacity: 0.9;
    transform: scale(1.03);
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
    background-color: rgba(255,255,255,0.9);
    border-radius: 10px;
    overflow: hidden;
  }

  th, td {
    border: 1px solid #d4af37;
    padding: 12px;
    text-align: center;
    color: #000; /* Letras pretas */
    font-weight: bold;
  }

  th {
    background-color: #d4af37;
    color: #000;
  }

  .resultado {
    margin-top: 20px;
    font-weight: bold;
    color: #f8f8f8;
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

  // Arredondar para múltiplos de 10
  p1 = Math.round(p1 / 10) * 10;
  p2 = Math.round(p2 / 10) * 10;
  p3 = Math.round(p3 / 10) * 10;
  p4 = Math.round(p4 / 10) * 10;

  const distribuido = p1 + p2 + p3 + p4;
  const sobra = restante - distribuido;

  let porquinhoFinal = porquinho + (sobra > 0 ? sobra : 0);

  premiosOriginais = { total, casa, porquinho: porquinhoFinal, p1, p2, p3, p4 };
  quintoAdicionado = false;

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

  // Decide valor e diluição padrão
  const valorQuinto = total < 1000 ? 40 : 80;
  const tirarPorPosicao = total < 1000 ? 10 : 20;

  const novoP1 = p1 - tirarPorPosicao;
  const novoP2 = p2 - tirarPorPosicao;
  const novoP3 = p3 - tirarPorPosicao;
  const novoP4 = p4 - tirarPorPosicao;

  const p5 = valorQuinto;

  if (p5 >= novoP4) {
    // Caso não seja possível salvar convencionalmente
    const confirmar = confirm("Não é possível salvar o quinto da forma convencional.\nDeseja salvar com valor fixo de R$ 40?");
    if (confirmar) {
      const novoP1_alt = p1 - 20;
      const novoP2_alt = p2 - 20;
      const p5_alt = 40;

      if (p5_alt >= p4) {
        alert("Mesmo com valor fixo, o quinto ficaria igual ou maior que o quarto. Ação cancelada.");
        return;
      }

      // Mantém o total distribuído igual
      const distribuidoAntigo = p1 + p2 + p3 + p4;
      const distribuidoNovo = novoP1_alt + novoP2_alt + p3 + p4 + p5_alt;
      const ajuste = distribuidoAntigo - distribuidoNovo;
      porquinho += ajuste;

      premiosOriginais = { total, casa, porquinho, p1: novoP1_alt, p2: novoP2_alt, p3, p4, p5: p5_alt };
      quintoAdicionado = true;
    } else {
      return;
    }
  } else {
    // Forma convencional aceita
    const distribuidoAntigo = p1 + p2 + p3 + p4;
    const distribuidoNovo = novoP1 + novoP2 + novoP3 + novoP4 + p5;
    const ajuste = distribuidoAntigo - distribuidoNovo;
    porquinho += ajuste;

    premiosOriginais = { total, casa, porquinho, p1: novoP1, p2: novoP2, p3: novoP3, p4: novoP4, p5 };
    quintoAdicionado = true;
  }

  mostrarTabela(premiosOriginais);
  document.getElementById('salvarQuinto').disabled = true;
  document.getElementById('desfazerQuinto').disabled = false;
});

document.getElementById('desfazerQuinto').addEventListener('click', () => {
  if (!quintoAdicionado) {
    alert("Nenhum quinto foi salvo ainda.");
    return;
  }
  document.getElementById('calcular').click();
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
