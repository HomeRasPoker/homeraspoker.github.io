<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Comanda Persistente</title>
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
  td.total { font-weight: bold; }
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
  </tr>
</table>

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

// Função para atualizar a tabela
function atualizarTabela(){
  // Limpar tabela (exceto cabeçalho)
  comandaTable.innerHTML = `
    <tr>
      <th>Nome</th>
      <th>Número</th>
      <th>Refri (R$5)</th>
      <th>Cerveja (R$15)</th>
      <th>Comida (R$25)</th>
      <th>Total</th>
    </tr>
  `;

  comandas.forEach((comanda,index)=>{
    const row = comandaTable.insertRow();
    
    // Nome
    const nomeCell = row.insertCell();
    nomeCell.textContent = comanda.nome;
    nomeCell.contentEditable = "true";
    nomeCell.onblur = () => {
      comanda.nome = nomeCell.textContent;
      salvarComandas();
    };

    // Número da comanda
    const numCell = row.insertCell();
    numCell.textContent = comanda.numero;

    // Refri
    const refriCell = row.insertCell();
    const btnRefri = document.createElement("button");
    btnRefri.textContent = "Refri";
    btnRefri.className = "btn-refri";
    btnRefri.onclick = () => adicionarValor(index,5);
    refriCell.appendChild(btnRefri);

    // Cerveja
    const cervejaCell = row.insertCell();
    const btnCerveja = document.createElement("button");
    btnCerveja.textContent = "Cerveja";
    btnCerveja.className = "btn-cerveja";
    btnCerveja.onclick = () => adicionarValor(index,15);
    cervejaCell.appendChild(btnCerveja);

    // Comida
    const comidaCell = row.insertCell();
    const btnComida = document.createElement("button");
    btnComida.textContent = "Comida";
    btnComida.className = "btn-comida";
    btnComida.onclick = () => adicionarValor(index,25);
    comidaCell.appendChild(btnComida);

    // Total
    const totalCell = row.insertCell();
    totalCell.textContent = `R$${comanda.total}`;
    totalCell.className = "total";
  });
}

// Adicionar valor à comanda
function adicionarValor(indice,valor){
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

// Inicializar
atualizarTabela();
</script>
</body>
</html>
