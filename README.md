<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Comandas Abertas e Fechadas</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<style>
body { font-family: Arial, sans-serif; background: #f5f5f5; padding: 20px; display:flex; flex-direction: column; align-items: center;}
h2 { color:#333; }
table { border-collapse: collapse; width: 90%; margin-bottom: 20px;}
th, td { border:1px solid #888; padding:6px 10px; text-align:center; }
th { background:#ffcc00; color:#000; }
td { color:#000; }
td[contenteditable="true"] { background:#fffae5; border:1px dashed #ffcc00; }
tr:nth-child(even){ background:#fdf5e6; }
tr:nth-child(odd){ background:#fff; }
button { padding: 4px 8px; margin:2px; border-radius:4px; cursor:pointer; border:none; font-weight:bold; }
.btn-refri { background:#4CAF50;color:#fff; }
.btn-cerveja { background:#2196F3;color:#fff; }
.btn-comida { background:#FF9800;color:#fff; }
.btn-export,.btn-add,.btn-geral,.btn-reset,.btn-fechar,.btn-apagar{ background:#ffcc00;color:#000; }
.btn-export:hover,.btn-add:hover,.btn-geral:hover,.btn-reset:hover,.btn-fechar:hover,.btn-apagar:hover{ background:#ffb300; }
.section-container { width: 95%; margin-bottom: 30px;}
</style>
</head>
<body>

<h2>Comandas Abertas</h2>
<div style="margin-bottom:10px;">
  <button class="btn-add" onclick="adicionarComanda()">➕ Adicionar Nova Comanda</button>
  <button class="btn-reset" onclick="resetarConsumoComandas()">🔄 Resetar Consumo</button>
  <button class="btn-geral" onclick="gerarRelatorioGeral()">📊 Gerar Relatório Geral</button>
  <button class="btn-apagar" onclick="apagarComandasFechadas()">🗑️ Apagar Comandas Fechadas</button>
</div>
<div class="section-container">
  <table id="comandaAbertasTable">
    <tr>
      <th>Nome</th><th>Número</th><th>Refri</th><th>Cerveja</th><th>Comida</th><th>Total</th><th>Finalizar</th><th>Exportar PNG</th>
    </tr>
  </table>
</div>

<h2>Comandas Fechadas</h2>
<div class="section-container">
  <table id="comandaFechadasTable">
    <tr>
      <th>Nome</th><th>Número</th><th>Refri</th><th>Cerveja</th><th>Comida</th><th>Total</th><th>Comprovante PNG</th>
    </tr>
  </table>
</div>

<script>
const produtosLista = [
  {nome:"Refri", valor:5},
  {nome:"Cerveja", valor:15},
  {nome:"Comida", valor:25}
];

let comandas = [];
if(localStorage.getItem("comandas")){
  comandas = JSON.parse(localStorage.getItem("comandas"));
} else {
  let nomes = ["João","Maria","Pedro","Ana","Lucas"];
  for(let i=0;i<10;i++){
    comandas.push({nome:nomes[Math.floor(Math.random()*nomes.length)], numero:i+1, total:0, produtos:{Refri:0,Cerveja:0,Comida:0}, fechada:false});
  }
  salvarComandas();
}

function salvarComandas(){ localStorage.setItem("comandas",JSON.stringify(comandas)); }

function resetarConsumoComandas(){
  if(confirm("Deseja realmente resetar o consumo de todas as comandas abertas?")){
    comandas.forEach(c=>{ if(!c.fechada){ c.total=0; c.produtos={Refri:0,Cerveja:0,Comida:0}; } });
    salvarComandas();
    atualizarTabelas();
  }
}

function atualizarTabelas(){
  const abertas = document.getElementById("comandaAbertasTable");
  const fechadas = document.getElementById("comandaFechadasTable");
  abertas.innerHTML=`<tr>
    <th>Nome</th><th>Número</th><th>Refri</th><th>Cerveja</th><th>Comida</th><th>Total</th><th>Finalizar</th><th>Exportar PNG</th>
  </tr>`;
  fechadas.innerHTML=`<tr>
    <th>Nome</th><th>Número</th><th>Refri</th><th>Cerveja</th><th>Comida</th><th>Total</th><th>Comprovante PNG</th>
  </tr>`;

  comandas.forEach((c,index)=>{
    const table = c.fechada ? fechadas : abertas;
    const row = table.insertRow();
    const nomeCell = row.insertCell(); nomeCell.textContent=c.nome; nomeCell.contentEditable=!c.fechada; nomeCell.onblur=()=>{c.nome=nomeCell.textContent; salvarComandas();}
    const numCell = row.insertCell(); numCell.textContent=c.numero; numCell.contentEditable=!c.fechada; numCell.onblur=()=>{ const n=parseInt(numCell.textContent); c.numero=isNaN(n)?c.numero:n; salvarComandas(); }

    produtosLista.forEach(prod=>{
      const cell=row.insertCell();
      if(!c.fechada){
        const btn=document.createElement("button");
        btn.textContent=prod.nome;
        btn.className=`btn-${prod.nome.toLowerCase()}`;
        btn.onclick=()=>{ c.total+=prod.valor; c.produtos[prod.nome]++; salvarComandas(); atualizarTabelas(); };
        cell.appendChild(btn);
      } else {
        cell.textContent=c.produtos[prod.nome];
      }
    });

    const totalCell=row.insertCell(); totalCell.textContent=`R$${c.total}`;

    if(!c.fechada){
      const finalizarCell=row.insertCell();
      const btnFinalizar=document.createElement("button");
      btnFinalizar.textContent="Finalizar";
      btnFinalizar.className="btn-fechar";
      btnFinalizar.onclick=()=>{ c.fechada=true; salvarComandas(); atualizarTabelas(); exportarComandaDetalhada(index); };
      finalizarCell.appendChild(btnFinalizar);

      const exportCell=row.insertCell();
      const btnExport=document.createElement("button");
      btnExport.textContent="Exportar PNG";
      btnExport.className="btn-export";
      btnExport.onclick=()=>exportarComandaDetalhada(index);
      exportCell.appendChild(btnExport);
    } else {
      const exportCell=row.insertCell();
      const btnExport=document.createElement("button");
      btnExport.textContent="PNG";
      btnExport.className="btn-export";
      btnExport.onclick=()=>exportarComandaDetalhada(index);
      exportCell.appendChild(btnExport);
    }
  });
}

function adicionarComanda(){
  const nome=prompt("Digite o nome da pessoa:");
  const numero=parseInt(prompt("Digite o número da comanda:"));
  if(nome && !isNaN(numero)){
    comandas.push({nome,numero,total:0,produtos:{Refri:0,Cerveja:0,Comida:0}, fechada:false});
    salvarComandas();
    atualizarTabelas();
  }
}

function exportarComandaDetalhada(index){
  const c=comandas[index];
  const div=document.createElement("div");
  div.style.padding="20px"; div.style.background="#fff"; div.style.border="2px solid #000"; div.style.width="300px"; div.style.fontFamily="Arial";
  div.innerHTML=`<h3>Comanda: ${c.nome} (#${c.numero})</h3>
    <p>Consumo Detalhado:</p>
    <ul>${produtosLista.map(p=>`<li>${p.nome}: ${c.produtos[p.nome]}</li>`).join('')}</ul>
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

// Função para gerar relatório geral
function gerarRelatorioGeral(){
  const fechadas = comandas.filter(c=>c.fechada);
  if(fechadas.length===0){ alert("Não há comandas fechadas para gerar relatório."); return; }

  let totalRefri=0, totalCerveja=0, totalComida=0, totalGeral=0;
  fechadas.forEach(c=>{
    totalRefri+=c.produtos.Refri;
    totalCerveja+=c.produtos.Cerveja;
    totalComida+=c.produtos.Comida;
    totalGeral+=c.total;
  });

  const div=document.createElement("div");
  div.style.padding="20px"; div.style.background="#fff"; div.style.border="2px solid #000"; div.style.width="350px"; div.style.fontFamily="Arial";
  div.innerHTML=`<h3>Relatório Geral</h3>
    <ul>
      <li>Total Refri: ${totalRefri} unidades</li>
      <li>Total Cerveja: ${totalCerveja} unidades</li>
      <li>Total Comida: ${totalComida} unidades</li>
    </ul>
    <p><b>Total Arrecadado: R$${totalGeral}</b></p>`;
  document.body.appendChild(div);
  html2canvas(div).then(canvas=>{
    const link=document.createElement("a");
    link.download="relatorio_geral.png";
    link.href=canvas.toDataURL();
    link.click();
    div.remove();
  });
}

// NOVO: Apagar todas as comandas fechadas
function apagarComandasFechadas(){
  if(confirm("Deseja realmente apagar todas as comandas fechadas? Esta ação não pode ser desfeita.")){
    comandas = comandas.filter(c=>!c.fechada);
    salvarComandas();
    atualizarTabelas();
  }
}

atualizarTabelas();
</script>
</body>
</html>
