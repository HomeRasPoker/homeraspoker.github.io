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
  }
  .comandaExport h3 { text-align: center; }
</style>
</head>
<body>

<h2>Comandas Persistentes Detalhadas</h2>
<button onclick="resetarComandas()">🔄 Resetar Todas as Comandas</button>
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

<script>
const nomes = ["Lucas","Ana","Pedro","Maria","João","Fernanda","Carlos","Beatriz","Rafael","Julia"];
const comandaTable = document.getElementById("comandaTable");

// Produtos e preços
const produtosLista = [
  {nome:"Refri", valor:5},
  {nome:"Cerveja", valor:15},
  {nome:"Comida", valor:25}
];

// Inicializar comandas
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

// Atualizar tabela
function atualizarTabela(){
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

    // Número
    const numCell = row.insertCell();
    numCell.textContent = c.numero;

    // Botões produtos
    produtosLista.forEach(prod=>{
      const cell = row.insertCell();
      const btn = document.createElement("button");
      btn.textContent = prod.nome;
      btn.className = `btn-${prod.nome.toLowerCase()}`;
      btn.onclick = () => {
        c.total += prod.valor;
        c.produtos[prod.nome]++;
        salvarComandas();
        atualizarTabela();
      };
      cell.appendChild(btn);
    });

    // Total
    const totalCell = row.insertCell();
    totalCell.textContent = `R$${c.total}`;
    totalCell.className = "total";

    // Exportar botão
    const exportCell = row.insertCell();
    const btnExport = document.createElement("button");
    btnExport.textContent = "Exportar PNG";
    btnExport.className = "btn-export";
    btnExport.onclick = () => exportarComandaDetalhada(index);
    exportCell.appendChild(btnExport);
  });
}

// Salvar localStorage
function salvarComandas(){
  localStorage.setItem("comandas", JSON.stringify(comandas));
}

// Resetar todas
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

// Exportar detalhado
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

// Inicializar tabela
atualizarTabela();
</script>
</body>
</html>
