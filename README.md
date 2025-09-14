<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Comanda Persistente</title>
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
  table { border-collapse: collapse; width: 90%; max-width: 900px; }
  th, td { border: 1px solid #aaa; padding: 8px; text-align: center; }
  th { background-color: #ffcc00; }
  td button { padding: 5px 10px; margin: 2px; cursor: pointer; border-radius: 5px; border: none; color: #fff; }
  .btn-refri { background-color: #4CAF50; }
  .btn-cerveja { background-color: #2196F3; }
  .btn-comida { background-color: #FF5722; }
  .btn-export { background-color: #795548; }
  td.total { font-weight: bold; }
  .comandaExport {
    display: none;
    padding: 20px;
    border-radius: 10px;
    background: #fff8e1;
    border: 2px solid #ffcc00;
    text-align: center;
    font-size: 18px;
    font-weight: bold;
  }
</style>
</head>
<body>

<h2>Comandas Persistentes</h2>
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

<!-- Div temporária para exportar a comanda -->
<div id="comandaExport" class="comandaExport"></div>

<script>
const nomes = ["Lucas", "Ana", "Pedro", "Maria", "João", "Fernanda", "Carlos", "Beatriz", "Rafael", "Julia"];
const comandaTable = document.getElementById("comandaTable");

// Carregar comandas do localStorage ou criar 10 iniciais
let comandas = [];
if(localStorage.getItem("comandas")){
  comandas = JSON.parse(localStorage.getItem("comandas"));
} else {
  for(let i=1;i<=10;i++){
    const randomNome = nomes[Math.floor(Math.random() * nomes.length)];
    comandas.push({nome: randomNome, numero:i, total:0});
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

  comandas.forEach((comanda,index)=>{
    const row = comandaTable.insertRow();
    
    // Nome editável
    const nomeCell = row.insertCell();
    nomeCell.textContent = comanda.nome;
    nomeCell.contentEditable = "true";
    nomeCell.onblur = () => {
      comanda.nome = nomeCell.textContent;
      salvarComandas();
    };

    // Número
    const numCell = row.insertCell();
    numCell.textContent = comanda.numero;

    // Botões de produtos
    const produtos = [
      {nome:"Refri", valor:5, classe:"btn-refri"},
      {nome:"Cerveja", valor:15, classe:"btn-cerveja"},
      {nome:"Comida", valor:25, classe:"btn-comida"}
    ];
    produtos.forEach(prod=>{
      const cell = row.insertCell();
      const btn = document.createElement("button");
      btn.textContent = prod.nome;
      btn.className = prod.classe;
      btn.onclick = () => adicionarValor(index, prod.valor);
      cell.appendChild(btn);
    });

    // Total
    const totalCell = row.insertCell();
    totalCell.textContent = `R$${comanda.total}`;
    totalCell.className = "total";

    // Exportar botão
    const exportCell = row.insertCell();
    const btnExport = document.createElement("button");
    btnExport.textContent = "Exportar PNG";
    btnExport.className = "btn-export";
    btnExport.onclick = () => exportarComanda(index);
    exportCell.appendChild(btnExport);
  });
}

// Adicionar valor
function adicionarValor(indice, valor){
  comandas[indice].total += valor;
  salvarComandas();
  atualizarTabela();
}

// Salvar no localStorage
function salvarComandas(){
  localStorage.setItem("comandas", JSON.stringify(comandas));
}

// Resetar todas as comandas
function resetarComandas(){
  if(confirm("Deseja zerar todas as comandas?")){
    comandas.forEach(c=>c.total=0);
    salvarComandas();
    atualizarTabela();
  }
}

// Exportar comanda individual
function exportarComanda(indice){
  const c = comandas[indice];
  const exportDiv = document.getElementById("comandaExport");
  exportDiv.style.display = "block";
  exportDiv.innerHTML = `
    <h3>Comanda Nº ${c.numero}</h3>
    <p>Nome: ${c.nome}</p>
    <p>Total: R$${c.total}</p>
  `;
  html2canvas(exportDiv).then(canvas=>{
    const link = document.createElement("a");
    link.download = `comanda_${c.numero}_${c.nome}.png`;
    link.href = canvas.toDataURL();
    link.click();
    exportDiv.style.display = "none";
  });
}

// Inicializar
atualizarTabela();
</script>
</body>
</html>
