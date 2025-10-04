<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Divisão de Premiação</title>
<style>
  body {
    font-family: 'Poppins', sans-serif;
    background: linear-gradient(135deg, #2b2b2b, #60492c);
    color: #fff;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 30px;
  }
  h1 {
    margin-bottom: 20px;
    color: #f0c674;
    text-shadow: 1px 1px 3px #000;
  }
  .container {
    background: #3a3a3a;
    border-radius: 15px;
    padding: 25px;
    width: 95%;
    max-width: 550px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.4);
  }
  input[type="number"] {
    width: 100%;
    padding: 10px;
    border: none;
    border-radius: 10px;
    margin-bottom: 20px;
    font-size: 1rem;
  }
  button {
    width: 100%;
    padding: 12px;
    margin-top: 10px;
    border: none;
    border-radius: 10px;
    background: #f0c674;
    color: #000;
    font-size: 1.1rem;
    cursor: pointer;
    transition: 0.3s;
  }
  button:hover {
    background: #ffdf91;
  }
  table {
    width: 100%;
    margin-top: 15px;
    border-collapse: collapse;
  }
  th, td {
    padding: 10px;
    text-align: center;
  }
  th {
    background: #60492c;
    color: #fff;
  }
  tr:nth-child(even) {
    background: #4a4a4a;
  }
  .extra-info {
    margin-top: 15px;
    text-align: center;
    font-weight: bold;
    color: #f0c674;
  }
</style>
</head>
<body>

  <h1>Divisão de Premiação</h1>
  <div class="container">
    <label for="premio">Valor total da premiação (R$):</label>
    <input type="number" id="premio" placeholder="Digite o valor total" min="0">
    <button onclick="calcularPremio()">Calcular Divisão</button>

    <table id="tabelaPremios" style="display:none;">
      <thead>
        <tr>
          <th>Colocação</th>
          <th>Valor (R$)</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>

    <div class="extra-info" id="infoExtras"></div>

    <button id="btnQuinto" style="display:none;" onclick="salvarQuinto()">💰 Salvar o Quinto</button>
  </div>

<script>
  let valores = {};
  let totalDivisivel = 0;

  function arredondaParaDez(valor) {
    return Math.round(valor / 10) * 10;
  }

  function calcularPremio() {
    const premioTotal = parseFloat(document.getElementById('premio').value);
    if (isNaN(premioTotal) || premioTotal <= 0) {
      alert('Por favor, insira um valor válido.');
      return;
    }

    // Cálculos iniciais
    const casa = premioTotal * 0.10;
    const porquinho = premioTotal * 0.05;
    let restante = premioTotal - casa - porquinho;

    // Divisão bruta
    let p1 = restante * 0.50;
    let p2 = restante * 0.25;
    let p3 = restante * 0.15;
    let p4 = restante * 0.10;

    // Arredonda e recolhe os quebrados menores que 10
    let totalQuebrados = 0;
    [p1, p2, p3, p4] = [p1, p2, p3, p4].map(v => {
      let arredondado = arredondaParaDez(v);
      if (Math.abs(v - arredondado) < 10) {
        totalQuebrados += Math.abs(v - arredondado);
        return arredondado;
      }
      return arredondado;
    });

    totalDivisivel = p1 + p2 + p3 + p4;

    valores = { casa, porquinho: porquinho + totalQuebrados, p1, p2, p3, p4, quinto: 0 };

    atualizarTabela();
  }

  function atualizarTabela() {
    const tabela = document.getElementById('tabelaPremios');
    const corpo = tabela.querySelector('tbody');
    corpo.innerHTML = '';

    const linhas = [
      ['1º Lugar', valores.p1],
      ['2º Lugar', valores.p2],
      ['3º Lugar', valores.p3],
      ['4º Lugar', valores.p4],
    ];

    if (valores.quinto > 0) linhas.push(['5º Lugar', valores.quinto]);

    linhas.forEach(([pos, val]) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `<td>${pos}</td><td>R$ ${val.toFixed(2)}</td>`;
      corpo.appendChild(tr);
    });

    tabela.style.display = 'table';
    document.getElementById('btnQuinto').style.display = 'block';
    document.getElementById('infoExtras').innerHTML =
      `🏠 Casa: R$ ${valores.casa.toFixed(2)}<br>🐷 Porquinho: R$ ${valores.porquinho.toFixed(2)}<br>💵 Total distribuído: R$ ${totalDivisivel.toFixed(2)}`;
  }

  function salvarQuinto() {
    if (valores.quinto > 0) {
      alert('O quinto já foi salvo!');
      return;
    }

    let totalReduzido = totalDivisivel - 100;
    valores.p1 -= 30;
    valores.p2 -= 30;
    valores.p3 -= 20;
    valores.p4 -= 20;
    valores.quinto = 100;
    totalDivisivel = totalReduzido + 100;

    atualizarTabela();
  }
</script>
</body>
</html>
