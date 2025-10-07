<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Comandas - Sistema Visual</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<style>
body { font-family: Arial, sans-serif; background: #f5f5f5; padding: 20px; display:flex; flex-direction: column; align-items: center; }
h2 { color:#333; margin-bottom:10px; }
table { border-collapse: collapse; width: 90%; margin-bottom: 20px; }
th, td { border:1px solid #888; padding:6px 10px; text-align:center; }
th { background:#ffcc00; color:#000; }
td { color:#000; }
td[contenteditable="true"] { background:#fffae5; border:1px dashed #ffcc00; }
tr:nth-child(even){ background:#fdf5e6; }
tr:nth-child(odd){ background:#fff; }
button { padding: 4px 8px; margin:2px; border-radius:4px; cursor:pointer; font-weight:bold; border:none; }
.btn-export,.btn-add,.btn-geral,.btn-reset,.btn-fechar,.btn-apagar,.btn-add-produto,.btn-editar-estoque,.btn-excluir-produto{ background:#ffcc00;color:#000; }
.btn-export:hover,.btn-add:hover,.btn-geral:hover,.btn-reset:hover,.btn-fechar:hover,.btn-apagar:hover,.btn-add-produto:hover,.btn-editar-estoque:hover,.btn-excluir-produto:hover{ background:#ffb300; }
.product-cell { font-weight:bold; color:#fff; padding:4px 8px; border-radius:4px; display:inline-block; margin:1px; }
.product-btn { padding:2px 4px; margin:1px; border-radius:3px; font-weight:bold; font-size:12px; color:#fff; border:none; cursor:pointer; }
.product-container { display:flex; flex-wrap:wrap; justify-content:center; gap:2px; }
.status-aberta { background:#81d4fa; }
.status-fechada { background:#ff8a80; }
.produtos-vendidos span { margin:0 2px; padding:2px 4px; border-radius:3px; color:#fff; font-weight:bold; display:inline-block; font-size:12px; }
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

<table id="comandaAbertasTable">
  <tr><th>Nome</th><th>Número</th><th>Total</th><th>Finalizar</th><th>PNG</th><th>Produtos</th><th>Vendidos</th></tr>
</table>

<h2>Comandas Fechadas</h2>
<table id="comandaFechadasTable">
  <tr><th>Nome</th><th>Número</th><th>Total</th><th>Comprovante PNG</th></tr>
</table>

<h2>Gestão de Estoque</h2>
<div style="margin-bottom:10px;">
  <button class="btn-add-produto" onclick="adicionarProduto()">➕ Adicionar Produto</button>
</div>
<div id="estoqueDiv"></div>

<script>
let coresProdutos = ["#4CAF50","#2196F3","#FF5722","#9C27B0","#FF9800","#795548","#607D8B","#FF4081"];
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
  const table=document.createElement("table");
  const header=table.insertRow();
  header.innerHTML="<th>Produto</th><th>Valor</th><th>Estoque</th><th>Editar</th><th>Excluir</th>";
  produtosLista.forEach((p,index)=>{
    const row=table.insertRow();
    row.style.background=coresProdutos[index % coresProdutos.length];
    const cellNome=row.insertCell(); cellNome.textContent=p.nome; cellNome.className="product-cell";
    row.insertCell().textContent=`R$${p.valor}`;
    const estoqueCell=row.insertCell(); estoqueCell.textContent=p.estoque;
    const editCell=row.insertCell();
    const btnEdit=document.createElement("button"); btnEdit.textContent="✏️"; btnEdit.className="btn-editar-estoque";
    btnEdit.onclick=()=>{
      const novoEstoque=parseInt(prompt(`Atualizar estoque de ${p.nome}:`,p.estoque));
      if(!isNaN(novoEstoque)){ p.estoque=novoEstoque; salvarComandas(); atualizarEstoqueDiv(); atualizarTabelas();}
    };
    editCell.appendChild(btnEdit);
    const delCell=row.insertCell();
    const btnDel=document.createElement("button"); btnDel.textContent="❌"; btnDel.className="btn-excluir-produto";
    btnDel.onclick=()=>{ if(confirm(`Excluir produto ${p.nome}?`)){ produtosLista.splice(index,1); salvarComandas(); atualizarEstoqueDiv(); atualizarTabelas(); }};
    delCell.appendChild(btnDel);
  });
  div.appendChild(table);
}

function resetarConsumoComandas(){
  if(confirm("Deseja realmente resetar o consumo de todas as comandas abertas?")){
    comandas.forEach(c=>{
      if(!c.fechada){
        for(let p in c.produtos){
          const prod = produtosLista.find(x=>x.nome===p);
          if(prod) prod.estoque += c.produtos[p];
        }
        c.total=0; c.produtos={};
      }
    });
    salvarComandas();
    atualizarTabelas();
    atualizarEstoqueDiv();
  }
}

function atualizarTabelas(){
  const abertas = document.getElementById("comandaAbertasTable");
  const fechadas = document.getElementById("comandaFechadasTable");
  abertas.innerHTML=`<tr><th>Nome</th><th>Número</th><th>Total</th><th>Finalizar</th><th>PNG</th><th>Produtos</th><th>Vendidos</th></tr>`;
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
      btnFinalizar.onclick=()=>{ c.fechada=true; salvarComandas(); atualizarTabelas(); exportarComandaDetalhada(index,true); };
      finalizarCell.appendChild(btnFinalizar);

      const pngCell=row.insertCell();
      const btnPNG=document.createElement("button"); btnPNG.textContent="📷 PNG"; btnPNG.className="btn-export";
      btnPNG.onclick=()=>exportarComandaDetalhada(index,false);
      pngCell.appendChild(btnPNG);

      const prodCell=row.insertCell(); 
      const divProdutos=document.createElement("div"); divProdutos.className="product-container";
      produtosLista.forEach((prod,pIndex)=>{
        const btn=document.createElement("button");
        btn.textContent=`${prod.nome} (+R$${prod.valor})`;
        btn.className="product-btn"; btn.style.background=coresProdutos[pIndex % coresProdutos.length];
        btn.disabled = prod.estoque<=0;
        btn.onclick=()=>{
          if(prod.estoque>0){
            c.produtos[prod.nome]=(c.produtos[prod.nome]||0)+1;
            c.total+=prod.valor;
            prod.estoque--;
            salvarComandas();
            atualizarTabelas();
            atualizarEstoqueDiv();
          }
        };
        divProdutos.appendChild(btn);
      });
      prodCell.appendChild(divProdutos);

      const vendidosDiv=row.insertCell();
      vendidosDiv.className="produtos-vendidos";
      for(let p in c.produtos){
        const span=document.createElement("span");
        const idx = produtosLista.findIndex(x=>x.nome===p);
        span.style.background=coresProdutos[idx % coresProdutos.length];
        span.textContent=`${p}: ${c.produtos[p]}`;
        vendidosDiv.appendChild(span);
      }

    } else {
      const pngCell=row.insertCell();
      const btnPNG=document.createElement("button"); btnPNG.textContent="📷 PNG"; btnPNG.className="btn-export";
      btnPNG.onclick=()=>exportarComandaDetalhada(index,true);
      pngCell.appendChild(btnPNG);
    }
  });
}

function adicionarComanda(){
  let nome=prompt("Nome do cliente:");
  if(!nome) return;
  let numero=prompt("Número da comanda:",comandas.length?Math.max(...comandas.map(c=>c.numero))+1:1);
  if(isNaN(numero) || numero==="") numero=comandas.length?Math.max(...comandas.map(c=>c.numero))+1:1;
  comandas.push({nome, numero:parseInt(numero), total:0, produtos:{}, fechada:false});
  salvarComandas();
  atualizarTabelas();
}

function adicionarProduto(){
  let nome=prompt("Nome do produto:");
  if(!nome) return;
  let valor=parseFloat(prompt("Valor do produto:"));
  if(isNaN(valor)) return alert("Valor inválido");
  let estoque=parseInt(prompt("Quantidade em estoque:"));
  if(isNaN(estoque)) return alert("Estoque inválido");
  produtosLista.push({nome, valor, estoque});
  salvarComandas();
  atualizarEstoqueDiv();
  atualizarTabelas();
}

function exportarComandaDetalhada(index,finalizada=false){
  const c=comandas[index];
  const div=document.createElement("div");
  div.style.padding="20px"; div.style.background="#fff"; div.style.border="2px solid #000"; div.style.width="350px"; div.style.fontFamily="Arial";
  div.innerHTML=`<h3>Comanda #${c.numero} - ${c.nome}</h3>`;
  div.innerHTML+=`<p>Status: ${finalizada ? "FINALIZADA E PAGA" : "ABERTA"}</p>`;
  div.innerHTML+="<ul>";
  for(let p in c.produtos){ div.innerHTML+=`<li>${p}: ${c.produtos[p]}</li>`; }
  div.innerHTML+="</ul>";
  div.innerHTML+=`<p><strong>Total: R$${c.total}</strong></p>`;
  document.body.appendChild(div);
  html2canvas(div).then(canvas=>{
    const win = window.open();
    win.document.body.appendChild(canvas);
    div.remove();
  });
}

function gerarRelatorioGeral(){
  const div=document.createElement("div");
  div.style.padding="20px"; div.style.background="#fff"; div.style.border="2px solid #000"; div.style.width="400px"; div.style.fontFamily="Arial";
  div.innerHTML="<h3>Relatório Geral</h3>";
  let totalGeral=0; let produtosTotais={};
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

function apagarComandasFechadas(){
  if(confirm("Deseja apagar todas as comandas fechadas?")){
    comandas=comandas.filter(c=>!c.fechada);
    salvarComandas();
    atualizarTabelas();
  }
}

atualizarEstoqueDiv();
atualizarTabelas();
</script>
</body>
</html>
