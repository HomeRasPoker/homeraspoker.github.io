<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Comandas Abertas e Fechadas com Estoque Dinâmico</title>
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
.btn-export,.btn-add,.btn-geral,.btn-reset,.btn-fechar,.btn-apagar,.btn-add-produto,.btn-editar-estoque,.btn-excluir-produto{ background:#ffcc00;color:#000; }
.btn-export:hover,.btn-add:hover,.btn-geral:hover,.btn-reset:hover,.btn-fechar:hover,.btn-apagar:hover,.btn-add-produto:hover,.btn-editar-estoque:hover,.btn-excluir-produto:hover{ background:#ffb300; }
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

<h2>Gestão de Estoque</h2>
<div style="margin-bottom:10px;">
  <button class="btn-add-produto" onclick="adicionarProduto()">➕ Adicionar Produto</button>
</div>
<div id="estoqueDiv"></div>

<div class="section-container">
  <table id="comandaAbertasTable">
    <tr>
      <th>Nome</th><th>Número</th><th>Total</th><th>Finalizar</th><th>Exportar PNG</th>
    </tr>
  </table>
</div>

<h2>Comandas Fechadas</h2>
<div class="section-container">
  <table id="comandaFechadasTable">
    <tr>
      <th>Nome</th><th>Número</th><th>Total</th><th>Comprovante PNG</th>
    </tr>
  </table>
</div>

<script>
let produtosLista = [];
if(localStorage.getItem("produtosLista")){
  produtosLista = JSON.parse(localStorage.getItem("produtosLista"));
} else {
  produtosLista = [
    {nome:"Refri", valor:5, estoque:20},
    {nome:"Cerveja", valor:15, estoque:15},
    {nome:"Comida", valor:25, estoque:10}
  ];
  localStorage.setItem("produtosLista",JSON.stringify(produtosLista));
}

let comandas = [];
if(localStorage.getItem("comandas")){
  comandas = JSON.parse(localStorage.getItem("comandas"));
} else {
  let nomes = ["João","Maria","Pedro","Ana","Lucas"];
  for(let i=0;i<10;i++){
    comandas.push({nome:nomes[Math.floor(Math.random()*nomes.length)], numero:i+1, total:0, produtos:{}, fechada:false});
  }
  salvarComandas();
}

function salvarComandas(){
  localStorage.setItem("comandas",JSON.stringify(comandas));
  localStorage.setItem("produtosLista",JSON.stringify(produtosLista));
}

function atualizarEstoqueDiv(){
  const div=document.getElementById("estoqueDiv");
  div.innerHTML="";
  produtosLista.forEach((p,index)=>{
    const span=document.createElement("span");
    span.innerHTML=`${p.nome} (R$${p.valor}): <input type="number" value="${p.estoque}" min="0" style="width:50px;" onchange="atualizarEstoque(${index},this.value)"> 
    <button class="btn-excluir-produto" onclick="excluirProduto(${index})">❌</button> `;
    div.appendChild(span); div.appendChild(document.createElement("br"));
  });
}

function atualizarEstoque(index,valor){
  produtosLista[index].estoque=parseInt(valor)||0;
  salvarComandas();
  atualizarTabelas();
}

function adicionarProduto(){
  const nome = prompt("Digite o nome do produto:");
  const valor = parseFloat(prompt("Digite o valor do produto:"));
  const estoque = parseInt(prompt("Digite a quantidade em estoque:"));
  if(nome && !isNaN(valor) && !isNaN(estoque)){
    produtosLista.push({nome, valor, estoque});
    salvarComandas();
    atualizarEstoqueDiv();
    atualizarTabelas();
  }
}

function excluirProduto(index){
  if(confirm(`Deseja realmente excluir o produto ${produtosLista[index].nome}?`)){
    produtosLista.splice(index,1);
    comandas.forEach(c=>{
      if(c.produtos[produtosLista[index]?.nome]) delete c.produtos[produtosLista[index]?.nome];
    });
    salvarComandas();
    atualizarEstoqueDiv();
    atualizarTabelas();
  }
}

function resetarConsumoComandas(){
  if(confirm("Deseja realmente resetar o consumo de todas as comandas abertas?")){
    comandas.forEach(c=>{ if(!c.fechada){ c.total=0; c.produtos={}; } });
    salvarComandas();
    atualizarTabelas();
  }
}

function atualizarTabelas(){
  const abertas = document.getElementById("comandaAbertasTable");
  const fechadas = document.getElementById("comandaFechadasTable");
  abertas.innerHTML=`<tr><th>Nome</th><th>Número</th><th>Total</th><th>Finalizar</th><th>Exportar PNG</th></tr>`;
  fechadas.innerHTML=`<tr><th>Nome</th><th>Número</th><th>Total</th><th>Comprovante PNG</th></tr>`;

  comandas.forEach((c,index)=>{
    const table = c.fechada ? fechadas : abertas;
    const row = table.insertRow();
    const nomeCell=row.insertCell(); nomeCell.textContent=c.nome; nomeCell.contentEditable=!c.fechada; nomeCell.onblur=()=>{c.nome=nomeCell.textContent; salvarComandas();}
    const numCell=row.insertCell(); numCell.textContent=c.numero; numCell.contentEditable=!c.fechada; numCell.onblur=()=>{ const n=parseInt(numCell.textContent); c.numero=isNaN(n)?c.numero:n; salvarComandas(); }
    const totalCell=row.insertCell(); totalCell.textContent=`R$${c.total}`;

    if(!c.fechada){
      const finalizarCell=row.insertCell();
      const btnFinalizar=document.createElement("button");
      btnFinalizar.textContent="Finalizar"; btnFinalizar.className="btn-fechar";
      btnFinalizar.onclick=()=>{ c.fechada=true; salvarComandas(); atualizarTabelas(); exportarComandaDetalhada(index); };
      finalizarCell.appendChild(btnFinalizar);

      const exportCell=row.insertCell();
      produtosLista.forEach(prod=>{
        const btn=document.createElement("button");
        btn.textContent=prod.nome;
        btn.disabled=prod.estoque<=0;
        btn.onclick=()=>{
          if(prod.estoque>0){
            c.produtos[prod.nome]=c.produtos[prod.nome]?c.produtos[prod.nome]+1:1;
            c.total+=prod.valor;
            prod.estoque--;
            salvarComandas(); atualizarTabelas();
          }
        };
        exportCell.appendChild(btn);
      });
    } else {
      row.insertCell(); // vazio para finalizar
      const exportCell=row.insertCell();
      const btnExport=document.createElement("button");
      btnExport.textContent="PNG"; btnExport.className="btn-export";
      btnExport.onclick=()=>exportarComandaDetalhada(index);
      exportCell.appendChild(btnExport);
    }
  });
}

function adicionarComanda(){
  const nome=prompt("Digite o nome da pessoa:");
  const numero=parseInt(prompt("Digite o número da comanda:"));
  if(nome && !isNaN(numero)){
    comandas.push({nome,numero,total:0,produtos:{}, fechada:false});
    salvarComandas();
    atualizarTabelas();
  }
}

function exportarComandaDetalhada(index){
  const c=comandas[index];
  const div=document.createElement("div");
  div.style.padding="20px"; div.style.background="#fff"; div.style.border="2px solid #000"; div.style.width="300px"; div.style.fontFamily="Arial";
  div.innerHTML=`<h3>Comanda: ${c.nome} (#${c.numero})</h3>${c.fechada?'<p style="color:red;font-weight:bold;">FINALIZADA E PAGA</p>':''}<ul>${Object.entries(c.produtos).map(([k,v])=>`<li>${k}: ${v}</li>`).join('')}</ul><p>Total: R$${c.total}</p>`;
  document.body.appendChild(div);
  html2canvas(div).then(canvas=>{
    const link=document.createElement("a");
    link.download=`comanda_${c.numero}.png`;
    link.href=canvas.toDataURL();
    link.click();
    div.remove();
  });
}

function gerarRelatorioGeral(){
  const div=document.createElement("div");
  div.style.padding="20px"; div.style.background="#fff"; div.style.border="2px solid #000"; div.style.width="400px"; div.style.fontFamily="Arial";
  div.innerHTML="<h3>Relatório Geral</h3>";
  let totalGeral=0;
  let produtosTotais={};
  comandas.filter(c=>c.fechada).forEach(c=>{
    div.innerHTML+=`<p>${c.nome} (#${c.numero}) - Total R$${c.total}</p>`;
    totalGeral+=c.total;
    for(let k in c.produtos){ produtosTotais[k]=(produtosTotais[k]||0)+c.produtos[k]; }
  });
  div.innerHTML+="<h4>Itens vendidos:</h4><ul>"+Object.entries(produtosTotais).map(([k,v])=>`<li>${k}: ${v}</li>`).join('')+"</ul>";
  div.innerHTML+=`<p><strong>Total Geral Arrecadado: R$${totalGeral}</strong></p>`;
  document.body.appendChild(div);
  html2canvas(div).then(canvas=>{
    const link=document.createElement("a");
    link.download="relatorio_geral.png";
    link.href=canvas.toDataURL();
    link.click();
    div.remove();
  });
}

atualizarEstoqueDiv();
atualizarTabelas();
</script>
</body>
</html>
