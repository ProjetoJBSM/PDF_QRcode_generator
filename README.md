# PDF_QRcode_generator

Gerador de PDF com QR Codes específicos a partir de uma fonte de dados

## 📋 Descrição

Aplicação web em Vue.js que gera PDFs com QR Codes únicos a partir de dados estruturados. Cada página do PDF contém um único QR Code correspondente a uma entrada dos dados fornecidos.

## ✨ Funcionalidades

- 📁 **Suporte a múltiplos formatos**: CSV e JSON
- 🎯 **Seleção de coluna**: Escolha qual coluna dos dados usar para gerar os QR Codes
- 📄 **Um QR Code por página**: Cada página do PDF contém um único QR Code
- 💻 **Processamento client-side**: Todos os dados são processados no navegador
- 🎨 **Interface intuitiva**: Design moderno e fácil de usar
- 🚀 **Sem servidor**: Funciona completamente no navegador

## 🚀 Como Usar

1. Acesse a aplicação: [https://projetojbsm.github.io/PDF_QRcode_generator/](https://projetojbsm.github.io/PDF_QRcode_generator/)

2. **Carregue seus dados**:
   - Clique na área de upload ou arraste um arquivo CSV ou JSON
   - Arquivos de exemplo estão disponíveis no repositório

3. **Selecione a coluna**:
   - Após carregar o arquivo, selecione qual coluna deseja usar para gerar os QR Codes
   - Uma pré-visualização dos primeiros 5 registros será exibida

4. **Gere o PDF**:
   - Clique em "Gerar PDF com QR Codes"
   - O PDF será baixado automaticamente com um QR Code por página

## 📊 Formato dos Dados

### CSV
```csv
id,nome,email,url
1,João Silva,joao@example.com,https://example.com/joao
2,Maria Santos,maria@example.com,https://example.com/maria
```

### JSON
```json
[
  {
    "id": "1",
    "nome": "João Silva",
    "email": "joao@example.com",
    "url": "https://example.com/joao"
  },
  {
    "id": "2",
    "nome": "Maria Santos",
    "email": "maria@example.com",
    "url": "https://example.com/maria"
  }
]
```

## 🛠️ Tecnologias Utilizadas

- **Vue.js 3**: Framework JavaScript progressivo
- **QRCode.js**: Geração de QR Codes
- **jsPDF**: Criação de documentos PDF
- **PapaParse**: Análise de arquivos CSV

## 📦 Arquivos de Exemplo

O repositório inclui arquivos de exemplo para teste:
- `sample-data.csv`: Exemplo de arquivo CSV
- `sample-data.json`: Exemplo de arquivo JSON

## 🌐 Hospedagem no GitHub Pages

Para hospedar sua própria versão:

1. Faça um fork deste repositório
2. Vá em Settings > Pages
3. Selecione a branch `main` como source
4. A aplicação estará disponível em: `https://seu-usuario.github.io/PDF_QRcode_generator/`

## 💡 Casos de Uso

- Gerar QR Codes para eventos (crachás, ingressos)
- Criar etiquetas com QR Codes para inventário
- Gerar códigos de acesso únicos
- Criar material impresso com links personalizados

## 🔒 Privacidade

Todos os dados são processados localmente no seu navegador. Nenhum dado é enviado para servidores externos.

## 📝 Licença

Este projeto é de código aberto e está disponível sob a licença MIT.

## 🤝 Contribuindo

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues ou pull requests.

## 📧 Contato

Para dúvidas ou sugestões, abra uma issue no GitHub.