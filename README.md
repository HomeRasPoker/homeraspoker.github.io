<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ras Poker – Premiação</title>
<link rel="icon" href="https://cdn-icons-png.flaticon.com/512/3141/3141129.png">
<style>
  * {
    box-sizing: border-box;
  }

  body {
    font-family: 'Poppins', sans-serif;
    background: radial-gradient(circle at center, #4e342e, #2e1e16);
    margin: 0;
    padding: 20px;
    display: flex;
    justify-content: center;
    align-items: flex-start;
    min-height: 100vh;
  }

  .container {
    width: 100%;
    max-width: 450px;
    background: linear-gradient(180deg, #3b2a1b, #24160f);
    border-radius: 20px;
    box-shadow: 0 8px 25px rgba(0,0,0,0.6);
    padding: 25px;
    text-align: center;
    color: #fff;
    position: relative;
  }

  h1 {
    color: #d4af37;
    text-shadow: 1px 1px 3px #000;
    font-size: 1.6rem;
    margin-bottom: 20px;
  }

  label {
    display: block;
    font-weight: 600;
    margin-bottom: 5px;
    color: #f5f5f5;
    font-size: 1rem;
  }

  input {
    width: 100%;
    padding: 12px;
    border-radius: 10px;
    border: 2px solid #d4af37;
    background-color: #fff;
    color: #000;
    font-size: 1rem;
    text-align: center;
    outline: none;
    margin-bottom: 15px;
  }

  button {
    width: 100%;
    padding: 14px;
    border: none;
    border-radius: 12px;
    font-size: 1rem;
    font-weight: bold;
    color: #fff;
    cursor: pointer;
    margin-bottom: 10px;
    transition: all 0.25s ease;
    box-shadow: 0 4px 8px rgba(0,0,0,0.3);
  }

  #calcular { background-color: #2e7d32; }
  #salvarQuinto { background-color: #c49102; }
  #desfazerQuinto { background-color: #a93226; }

  button:hover {
    transform: translateY(-2px);
    opacity: 0.95;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
    background-color: rgba(255,255,255,0.95);
    border-radius: 12px;
    overflow: hidden;
  }

  th, td {
    border: 1px solid #d4af37;
    padding: 10px;
    text-align: center;
    color: #000;
    font-weight: 600;
  }

  th {
    background-color: #d4af37;
    color: #000;
  }

  .resultado p {
    color: #f5f5f5;
    font-weight: 600;
    margin-top: 10px;
  }

  #mensagem {
    display: none;
    position: absolute;
    top: 10%;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(212,175,55,0.95);
    color: #000;
    font-weight: bold;
    padding: 12px 20px;
    border-radius: 10px;
    box-shadow: 0 0 15px rgba(212,175,55,0.8);
    animation: fadeInOut 2s ease-in-out;
    z-index: 10;
  }

  @keyframes fadeInOut {
    0% { opacity: 0; transform: translate(-50%, -10px); }
    10% { opacity: 1; transform: translate(-50%, 0); }
    90% { opacity: 1; }
    100% { opacity: 0; transform: translate(-50%, -10px); }
  }

  @media (max-width: 480px) {
    body {
      padding: 10px;
    }
    .container {
      padding: 20px;
      border-radius: 15px;
    }
    h1 {
      font-size: 1.4rem;
    }
    button {
      font-size: 0.95rem;
      padding: 12px;
    }
    input {
      font-size: 0.95rem;
    }
  }
</style>
</head>
<body>

<div class="container">
  <div id="mensagem">Quinto salvo com sucesso!</div>
  <h1>Ras Poker – Premiação</h1>
  
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

  const casa = total * 0.10;
  const porquinho = total * 0.05;
  let restante = total - casa - porquinho;

  let p1 = restante * 0.50;
  let p2 = restante * 0.25;
  let p3 = restante * 0.15;
  let p4 = restante * 0.10;

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
  const valorQuinto = total < 1000 ? 40 : 80;
  const tirarPorPosicao = total < 1000 ? 10 : 20;

  const novoP1 = p1 - tirarPorPosicao;
  const novoP2 = p2 - tirarPorPosicao;
  const novoP3 = p3 - tirarPorPosicao;
  const novoP4 = p4 - tirarPorPosicao;
  const p5 = valorQuinto;

  if (p5 >= novoP4) {
    const confirmar = confirm("Não é possível salvar o quinto da forma convencional.\nDeseja salvar com valor fixo de R$ 40?");
    if (confirmar) {
      const novoP1_alt = p1 - 20;
      const novoP2_alt = p2 - 20;
      const p5_alt = 40;

      if (p5_alt >= p4) {
        alert("Mesmo com valor fixo, o quinto ficaria igual ou maior que o quarto. Ação cancelada.");
        return;
      }

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
    const distribuidoAntigo = p1 + p2 + p3 + p4;
    const distribuidoNovo = novoP1 + novoP2 + novoP3 + novoP4 + p5;
    const ajuste = distribuidoAntigo - distribuidoNovo;
    porquinho += ajuste;

    premiosOriginais = { total, casa, porquinho, p1: novoP1, p2: novoP2, p3: novoP3, p4: novoP4, p5 };
    quintoAdicionado = true;
  }

  mostrarTabela(premiosOriginais);
  mostrarMensagem("Quinto salvo com sucesso!");
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

function mostrarMensagem(texto) {
  const msg = document.getElementById("mensagem");
  msg.innerText = texto;
  msg.style.display = "block";
  setTimeout(() => { msg.style.display = "none"; }, 2000);
}
</script>
</body>
</html>
