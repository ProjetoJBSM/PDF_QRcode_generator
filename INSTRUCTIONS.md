# 📖 Instruções de Uso

## Passo a Passo

### 1️⃣ Preparar seus dados

Prepare seus dados em um dos formatos suportados:

**Formato CSV:**
```csv
id,nome,email,url
1,João Silva,joao@example.com,https://exemplo.com/joao
2,Maria Santos,maria@example.com,https://exemplo.com/maria
```

**Formato JSON:**
```json
[
  {
    "id": "1",
    "nome": "João Silva",
    "email": "joao@example.com",
    "url": "https://exemplo.com/joao"
  },
  {
    "id": "2",
    "nome": "Maria Santos",
    "email": "maria@example.com",
    "url": "https://exemplo.com/maria"
  }
]
```

### 2️⃣ Acessar a aplicação

Acesse: [https://projetojbsm.github.io/PDF_QRcode_generator/](https://projetojbsm.github.io/PDF_QRcode_generator/)

### 3️⃣ Carregar o arquivo

- **Clique** na área de upload, ou
- **Arraste e solte** seu arquivo CSV ou JSON

### 4️⃣ Selecionar a coluna

Após o arquivo ser carregado:
1. Um dropdown aparecerá com todas as colunas disponíveis
2. Selecione qual coluna deseja usar para gerar os QR Codes
3. Uma pré-visualização dos primeiros 5 registros será exibida

### 5️⃣ Gerar o PDF

1. Clique no botão "🎯 Gerar PDF com QR Codes"
2. Aguarde o processamento (cada QR Code leva aproximadamente 0.1 segundos)
3. O PDF será baixado automaticamente

## 💡 Dicas

### Qualidade dos QR Codes
- Os QR Codes são gerados com alta correção de erros (nível H)
- Cada QR Code tem 512x512 pixels de resolução
- Tamanho no PDF: 100mm x 100mm (ideal para impressão)

### Conteúdo dos QR Codes
Os QR Codes podem conter:
- URLs (links para sites)
- Texto simples
- E-mails
- Números de telefone
- Qualquer informação textual

### Limites
- Não há limite teórico de registros
- Para arquivos muito grandes (>1000 registros), o processamento pode levar alguns minutos
- Cada página do PDF contém exatamente 1 QR Code

## ❓ Casos de Uso

### 1. Crachás de Eventos
```csv
id,nome,empresa,link_perfil
1,João Silva,Empresa A,https://linkedin.com/in/joao
2,Maria Santos,Empresa B,https://linkedin.com/in/maria
```
Selecione a coluna `link_perfil` para gerar QR Codes que direcionam ao perfil profissional.

### 2. Etiquetas de Produtos
```csv
codigo,produto,url_manual
001,Produto A,https://manuais.empresa.com/produto-a
002,Produto B,https://manuais.empresa.com/produto-b
```
Selecione `url_manual` para criar etiquetas que direcionam ao manual do produto.

### 3. Ingressos
```csv
ticket_id,evento,validacao_url
T001,Evento Musical,https://validacao.site/T001
T002,Evento Musical,https://validacao.site/T002
```
Selecione `validacao_url` para gerar ingressos com QR Codes únicos.

### 4. Links de Pagamento
```csv
cliente_id,cliente_nome,link_pagamento
C001,Cliente A,https://pagamento.site/checkout/C001
C002,Cliente B,https://pagamento.site/checkout/C002
```
Selecione `link_pagamento` para criar PDFs com links de pagamento personalizados.

## 🔧 Solução de Problemas

### O arquivo não está sendo carregado
- Verifique se o formato é CSV ou JSON
- Certifique-se de que o CSV tem cabeçalhos (primeira linha)
- Verifique se o JSON é um array válido

### Os QR Codes não aparecem no PDF
- Verifique sua conexão com internet (bibliotecas são carregadas via CDN)
- Tente recarregar a página
- Verifique se há mensagens de erro no console do navegador

### Alguma coluna não aparece no dropdown
- Verifique se a primeira linha do CSV contém os nomes das colunas
- Certifique-se de que todas as linhas têm o mesmo número de colunas

### O PDF está muito grande
- O tamanho do PDF aumenta com o número de QR Codes
- Para grandes volumes, considere dividir em múltiplos arquivos

## 🔒 Privacidade e Segurança

- **Processamento local**: Todos os dados são processados no seu navegador
- **Sem envio de dados**: Nenhum dado é enviado para servidores externos
- **Código aberto**: O código está disponível para auditoria no GitHub

## 📞 Suporte

Se encontrar problemas ou tiver sugestões:
1. Abra uma [issue no GitHub](https://github.com/ProjetoJBSM/PDF_QRcode_generator/issues)
2. Descreva o problema detalhadamente
3. Inclua exemplos de dados (se possível) ou screenshots

## 🎓 Recursos Adicionais

- [Documentação Vue.js](https://vuejs.org/)
- [Especificação QR Code](https://www.qrcode.com/en/about/)
- [jsPDF Documentation](https://github.com/parallax/jsPDF)
