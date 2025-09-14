<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Comandas - Evento</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; background: #f5f5f5; display: flex; flex-direction: column; align-items: center; padding: 20px;}
    h2 { color: #333; }
    table { border-collapse: collapse; width: 90%; margin-bottom: 20px;}
    th, td { border: 1px solid #888; padding: 6px 10px; text-align: center; }
    th { background: #ffcc00; color: #000; }
    td { background: #fff; color: #000; }
    td[contenteditable="true"] { background: #fffae5; border: 1px dashed #ffcc00; }
    button { padding: 4px 10px; margin: 2px; border-radius: 4px; cursor: pointer; border: none; font-weight: bold; }
    .btn-refri { background:#4CAF50; color:#fff; }
    .btn-cerveja { background:#2196F3; color:#fff; }
    .btn-comida { background:#FF9800; color:#fff; }
    .btn-export, .btn-add, .btn-geral { background:#ffcc00; color:#000; }
    .btn-export:hover, .btn-add:hover, .btn-geral:hover { background:#ffb300; }
  </style>
</head>
<body>

<h2>Comandas do Evento</h2>
<div style="margin-bottom:10px;">
  <button class="btn-add" onclick="adicionarComanda()">➕ Adicionar Nova Comanda</button>
  <button class="btn-geral" onclick="exportarComandasGerais()">📄 Exportar Relatório Geral</button>
</div>

<table id="comandaTable">
  <tr>
    <th>Nome</th>
    <th>Número</th>
    <th>Refri (R$5)</th>
    <th>Cerveja (R$15)</th>
    <th>Comida (R$25)</th>
    <th>Total</th>
    <th>Exportar PNG</th>
  </tr>
</table>

<script>
const produtosLista = [
  {nome:"Refri", valor:5},
  {nome:"Cerveja", valor:15},
  {nome:"Comida", valor:25}
];

// Nomes aleatórios iniciais
let nomes = ["João","Maria","Pedro","Ana","Lucas"];
let comandas = [];

// Carregar do localStorage
if(localStorage.getItem("comandas")){
  comandas = JSON.parse(localStorage.getItem("comandas"));
}else{
  for(let i=0;i<10;i++){
    comandas.push({
      nome: nomes[Math.floor(Math.random()*nomes.length)],
      numero: i+1,
      total:0,
      produtos:{Refri:0,Cerveja:0,Comida:0}
    });
  }
  salvarComandas();
}

// Função para salvar
function salvarComandas(){
  localStorage.setItem("comandas", JSON.stringify(comandas));
}

// Atualizar tabela completa
function atualizarTabela(){
  const table = document.getElementById("comandaTable");
  table.innerHTML = `<tr>
    <th>Nome</th>
    <th>Número</th>
    <th>Refri (R$5)</th>
    <th>Cerveja (R$15)</th>
    <th>Comida (R$25)</th>
    <th>Total</th>
    <th>Exportar PNG</th>
  </tr>`;
  comandas.forEach(c=> adicionarLinhaTabela(c));
}

// Adicionar linha individual
function adicionarLinhaTabela(c){
  const table = document.getElementById("comandaTable");
  const row = table.insertRow();

  const nomeCell = row.insertCell();
  nomeCell.textContent = c.nome;
  nomeCell.contentEditable="true";
  nomeCell.onblur = () => { c.nome=nomeCell.textContent; salvarComandas(); };

  const numCell = row.insertCell();
  numCell.textContent = c.numero;
  numCell.contentEditable="true";
  numCell.onblur = () => { const n=parseInt(numCell.textContent); c.numero=isNaN(n)?c.numero:n; salvarComandas(); };

  produtosLista.forEach(prod=>{
    const cell = row.insertCell();
    const btn = document.createElement("button");
    btn.textContent = prod.nome;
    btn.className = `btn-${prod.nome.toLowerCase()}`;
    btn.addEventListener("click", ()=>{
      c.total += prod.valor;
      c.produtos[prod.nome]++;
      salvarComandas();
      atualizarTabela();
    });
    cell.appendChild(btn);
  });

  const totalCell = row.insertCell();
  totalCell.textContent = `R$${c.total}`;

  const exportCell = row.insertCell();
  const btnExport = document.createElement("button");
  btnExport.textContent="Exportar PNG";
  btnExport.className="btn-export";
  btnExport.addEventListener("click", ()=>exportarComandaDetalhada(comandas.indexOf(c)));
  exportCell.appendChild(btnExport);
}

// Adicionar nova comanda
function adicionarComanda(){
  const nome = prompt("Digite o nome da pessoa:");
  const numero = parseInt(prompt("Digite o número da comanda:"));
  if(nome && !isNaN(numero)){
    const novaComanda = {nome,numero,total:0,produtos:{Refri:0,Cerveja:0,Comida:0}};
    comandas.push(novaComanda);
    salvarComandas();
    adicionarLinhaTabela(novaComanda);
  }
}

// Exportar comanda detalhada em PNG
function exportarComandaDetalhada(index){
  const c = comandas[index];
  const div = document.createElement("div");
  div.style.padding="20px"; div.style.background="#fff"; div.style.border="2px solid #000"; div.style.width="300px"; div.style.fontFamily="Arial";
  div.innerHTML = `<h3>Comanda: ${c.nome} (#${c.numero})</h3>
  <p>Consumo Detalhado:</p>
  <ul>
    ${produtosLista.map(p=>`<li>${p.nome}: ${c.produtos[p.nome]}</li>`).join('')}
  </ul>
  <p><b>Total: R$${c.total}</b></p>`;
  document.body.appendChild(div);
  html2canvas(div).then(canvas=>{
    const link=document.createElement("a");
    link.download=`comanda_${c.nome}.png`;
    link.href=canvas.toDataURL();
    link.click();
    div.remove();
  });
}

// Exportar todas as comandas em relatório geral
function exportarComandasGerais(){
  const div = document.createElement("div");
  div.style.padding="20px"; div.style.background="#fff"; div.style.border="2px solid #000"; div.style.width="400px"; div.style.fontFamily="Arial";
  div.innerHTML = `<h3>Relatório Geral</h3>
  <table border="1" style="border-collapse: collapse; width:100%;">
    <tr><th>Nome</th><th>Número</th><th>Refri</th><th>Cerveja</th><th>Comida</th><th>Total</th></tr>
    ${comandas.map(c=>`<tr>
      <td>${c.nome}</td>
      <td>${c.numero}</td>
      <td>${c.produtos.Refri}</td>
      <td>${c.produtos.Cerveja}</td>
      <td>${c.produtos.Comida}</td>
      <td>R$${c.total}</td>
    </tr>`).join('')}
  </table>`;
  document.body.appendChild(div);
  html2canvas(div).then(canvas=>{
    const link=document.createElement("a");
    link.download="relatorio_geral.png";
    link.href=canvas.toDataURL();
    link.click();
    div.remove();
  });
}

// Inicializar tabela
atualizarTabela();
</script>

</body>
</html>
