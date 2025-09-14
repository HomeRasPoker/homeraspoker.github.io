<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Comanda Detalhada</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<style>
  body {
    font-family: Arial, sans-serif;
    background: #f5f5f5;
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 30px;
  }
  h2 { color: #333; }
  .topo { margin-bottom: 15px; }
  .topo button { padding: 10px 15px; margin-right: 5px; border-radius: 5px; border: none; cursor: pointer; color: #fff; font-weight: bold; }
  .btn-reset { background-color: #f44336; }
  .btn-export-geral { background-color: #607d8b; }
  .btn-add { background-color: #4CAF50; }
  table { border-collapse: collapse; width: 90%; max-width: 900px; margin-bottom: 20px; }
  th, td { border: 1px solid #aaa; padding: 8px; text-align: center; }
  th { background-color: #ffcc00; }
  td button { padding: 5px 10px; margin: 2px; cursor: pointer; border-radius: 5px; border: none; color: #fff; }
  .btn-refri { background-color: #4CAF50; }
  .btn-cerveja { background-color: #2196F3; }
  .btn-comida { background-color: #FF5722; }
  .btn-export { background-color: #795548; }
  td.total { font-weight: bold; }
  td[contenteditable="true"] { background: rgba(255,236,179,0.3); border: 1px dashed #ffcc00; }
  .comandaExport {
    display: none;
    padding: 20px;
    border-radius: 10px;
    background: #fff8e1;
    border: 2px solid #ffcc00;
    text-align: left;
    font-size: 18px;
    font-weight: bold;
    max-width: 700px;
  }
  .comandaExport h3 { text-align: center; }
</style>
</head>
<body>

<h2>Comandas Detalhadas</h2>
<div class="topo">
  <button class="btn-reset" onclick="resetarComandas()">🔄 Resetar Todas as Comandas</button>
  <button class="btn-add" onclick="adicionarComanda()">➕ Adicionar Comanda</button>
  <button class="btn-export-geral" onclick="exportarRelatorioGeral()">📋 Exportar Relatório Geral</button>
</div>

<table id="comandaTable">
  <tr>
    <th>Nome</th>
    <th>Número</th>
    <th>Refri (R$5)</th>
    <th>Cerveja (R$15)</th>
    <th>Comida (R$25)</th>
    <th>Total</th>
    <th>Exportar</th>
  </tr>
</table>

<div id="comandaExport" class="comandaExport"></div>
<div id="comandaExportGeral" class="comandaExport"></div>

<script>
const nomes = ["Lucas","Ana","Pedro","Maria","João","Fernanda","Carlos","Beatriz","Rafael","Julia"];
const produtosLista = [
  {nome:"Refri", valor:5},
  {nome:"Cerveja", valor:15},
  {nome:"Comida", valor:25}
];

let comandas = [];
if(localStorage.getItem("comandas")){
  comandas = JSON.parse(localStorage.getItem("comandas"));
} else {
  for(let i=1;i<=10;i++){
    const randomNome = nomes[Math.floor(Math.random() * nomes.length)];
    comandas.push({
      nome: randomNome,
      numero:i,
      total:0,
      produtos: { Refri:0, Cerveja:0, Comida:0 }
    });
  }
  localStorage.setItem("comandas", JSON.stringify(comandas));
}

function atualizarTabela(){
  const comandaTable = document.getElementById("comandaTable");
  comandaTable.innerHTML = `
    <tr>
      <th>Nome</th>
      <th>Número</th>
      <th>Refri (R$5)</th>
      <th>Cerveja (R$15)</th>
      <th>Comida (R$25)</th>
      <th>Total</th>
      <th>Exportar</th>
    </tr>
  `;

  comandas.forEach((c,index)=>{
    const row = comandaTable.insertRow();

    // Nome editável
    const nomeCell = row.insertCell();
    nomeCell.textContent = c.nome;
    nomeCell.contentEditable = "true";
    nomeCell.onblur = () => { c.nome = nomeCell.textContent; salvarComandas(); };

    // Número editável
    const numCell = row.insertCell();
    numCell.textContent = c.numero;
    numCell.contentEditable = "true";
    numCell.onblur = () => { 
      const n = parseInt(numCell.textContent);
      c.numero = isNaN(n)?c.numero:n;
      salvarComandas();
    };

    // Botões produtos
    produtosLista.forEach(prod=>{
      const cell = row.insertCell();
      const btn = document.createElement("button");
      btn.textContent = prod.nome;
      btn.className = `btn-${prod.nome.toLowerCase()}`;
      btn.addEventListener("click", () => {
        c.total += prod.valor;
        c.produtos[prod.nome]++;
        salvarComandas();
        atualizarTabela();
      });
      cell.appendChild(btn);
    });

    // Total
    const totalCell = row.insertCell();
    totalCell.textContent = `R$${c.total}`;
    totalCell.className = "total";

    // Exportar botão individual
    const exportCell = row.insertCell();
    const btnExport = document.createElement("button");
    btnExport.textContent = "Exportar PNG";
    btnExport.className = "btn-export";
    btnExport.addEventListener("click", () => exportarComandaDetalhada(index));
    exportCell.appendChild(btnExport);
  });
}

function salvarComandas(){
  localStorage.setItem("comandas", JSON.stringify(comandas));
}

function resetarComandas(){
  if(confirm("Deseja zerar todas as comandas?")){
    comandas.forEach(c=>{
      c.total=0;
      c.produtos = {Refri:0, Cerveja:0, Comida:0};
    });
    salvarComandas();
    atualizarTabela();
  }
}

// Adicionar nova comanda
function adicionarComanda(){
  const nome = prompt("Digite o nome da pessoa:");
  const numero = parseInt(prompt("Digite o número da comanda:"));
  if(nome && !isNaN(numero)){
    comandas.push({nome, numero, total:0, produtos:{Refri:0, Cerveja:0, Comida:0}});
    salvarComandas();
    atualizarTabela();
  }
}

// Exportar detalhado individual
function exportarComandaDetalhada(indice){
  const c = comandas[indice];
  const exportDiv = document.getElementById("comandaExport");
  exportDiv.style.display = "block";
  exportDiv.innerHTML = `<h3>Comanda Nº ${c.numero}</h3>
    <p>Nome: ${c.nome}</p>
    <p>Produtos consumidos:</p>
    <ul>
      ${produtosLista.map(p=>`<li>${p.nome}: ${c.produtos[p.nome]}x (R$${p.valor * c.produtos[p.nome]})</li>`).join("")}
    </ul>
    <p>Total: <strong>R$${c.total}</strong></p>
  `;
  html2canvas(exportDiv).then(canvas=>{
    const link = document.createElement("a");
    link.download = `comanda_${c.numero}_${c.nome}.png`;
    link.href = canvas.toDataURL();
    link.click();
    exportDiv.style.display = "none";
  });
}

// Exportar relatório geral
function exportarRelatorioGeral(){
  const exportDiv = document.getElementById("comandaExportGeral");
  exportDiv.style.display = "block";
  exportDiv.innerHTML = `<h3>Relatório Geral das Comandas</h3>
    ${comandas.map(c=>`
      <p>Comanda Nº ${c.numero} - ${c.nome}</p>
      <ul>
        ${produtosLista.map(p=>`<li>${p.nome}: ${c.produtos[p.nome]}x (R$${p.valor*c.produtos[p.nome]})</li>`).join("")}
      </ul>
      <p>Total: R$${c.total}</p>
      <hr>
    `).join("")}
  `;
  html2canvas(exportDiv).then(canvas=>{
    const link = document.createElement("a");
    link.download = `relatorio_geral_comandas.png`;
    link.href = canvas.toDataURL();
    link.click();
    exportDiv.style.display = "none";
  });
}

// Inicializar tabela
atualizarTabela();
</script>
</body>
</html>
