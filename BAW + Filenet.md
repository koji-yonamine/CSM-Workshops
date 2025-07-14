# Sicoob - BAW and Filenet

## Acessos Importantes 

**Cloud pak:**  
https://<SEU PATH>/zen/#/homepage

**Filenet Navigator – Ver documentos:**  
https://<SEU PATH>/icn/navigator/

**Filenet Navigator – Configurações:**  
https://<SEU PATH>/icn/navigator/?desktop=admin

**Filenet ACCE – Configurações:**  
https://<SEU PATH>/cpe/acce/

---

## Navigator – Diretórios – Criar pasta Contratos:

Cloud Pak -> Automação de Negócios (Business Automation)  
Workflow

---

## Conexão com o servidor 

**Servidores:**

Adicionar novo Servidor:

- Nome: qualquer  
- Servidor: Enterprise Content Management  
- Host: `<SEU PATH>`  
- Porta: `9443`  
- Caminho: `/cmis/openfncmis_wlp/services`  
- Servidor Seguro: `TRUE`  
- Repositório: `<SEU OBJECT STORE>`  
- Usuário: `<SEU USER>`  
- Senha: `<SUA SENHA>`


---

## Configuração das telas e serviços

**Serviço de autorização de consulta:**

**Incluir Tarefa de Script:** 
```javascript

ServiceFlow [
  tw.local.outCmisQuery = tw.local.inCmisQuery;
]

````
Testar Ligação;

---

## Importação

- CreatePDF.jar (importar)
- CreatePDF (importar)

---  

## Dados:

```javascript

dadosClienteBO [
  nome (String)
  cpf (String)
  telefone (String)
]

dadosContratoBO [
  agencia (String)
  codigo (String)
]

ECMContentStream [ 
  contentLength (integer)
  mimeType (string)
  fileName (string)
  content (any)
]

ECMProperty [ 
  objectTypeId (ECMID)
  value (any)
]
```

---

## Serviços:
**DFBCreatePDF**

```javascript
Tarefa de Serviço [

  Implementação { 
    CreatePDF
  }

  Operação {
    generatePDF
  }
]

Variáveis [
  Saída {
    Output1 (String)
  }
  Privado {
    stringEntrada (string)
  }
]
```

---

**SaveFileNet**

```Javascript
Tarefa de Script [

Script {
// Conteúdo PDF já em base64
var varBase64PDF = tw.local.pdfBase64;

// Decodifica a base64 para array de bytes
var bytesValue = Packages.java.util.Base64.getDecoder().decode(varBase64PDF);

// Cria um ECMContentStream para armazenar o conteúdo PDF
tw.local.pdfStream = new tw.object.ECMContentStream();
tw.local.pdfStream.contentLength = bytesValue.length;
tw.local.pdfStream.mimeType = "application/pdf";
tw.local.pdfStream.content = varBase64PDF;
tw.local.pdfStream.fileName = "Form_" + tw.system.currentTask.id + ".pdf";
}
]

---

Tarefa de Integração [
Implementação {
- Integração > ECM_Server
- Operação > Criar Documento
}

Mapeamento de entrada {
"cmis:document"
"/Contratos/"
"Form_" + tw.system.currentTask.id + ".pdf"

tw.local.pdfStream
}

Variáveis [
Entrada {
  pdfBase64 (string)
  formJSON (string)
}
Privado {
  pdfStream (ECMContentStream)
}

]]
```

---

**SaveFileNetJSON**

```Javascript
Tarefa de Script [
Script {
var value = tw.local.formJSON;
var bytesValue = new Packages.java.lang.String(value).getBytes("UTF-8");
tw.local.pdfStream = new tw.object.ECMContentStream();
tw.local.pdfStream.contentLength = bytesValue.length;
tw.local.pdfStream.mimeType = "application/json";
tw.local.pdfStream.content = Packages.javax.xml.bind.DatatypeConverter.printBase64Binary(bytesValue);
}

]

---

Tarefa de Integração [
Implementação {
- Integração > ECM_Server
- Operação > Criar Documento
}

Mapeamento de entrada {
"cmis:document"
"/Contratos/"
"Form_" + tw.system.currentTask.id + ".json"

tw.local.pdfStream
}

Variáveis [
Entrada {
  pdfBase64 (string)
  formJSON (string)
}
Privado {
  pdfStream (ECMContentStream)
}
]

]
```

**TratamentoFormulario**
```Javascript
Tarefa de Script [
Script 1 {
var formData = {
  contrato: tw.local.dadosContratoBO,
  cliente: tw.local.dadosClienteBO
};

tw.local.formJSON = JSON.stringify(formData);

}

Script 2 {
tw.local.stringPDF = 
  "Dados do Cliente" + "\n" +
  "Nome tomador: " + tw.local.dadosClienteBO.nome + "\n" +
  "CPF: " + tw.local.dadosClienteBO.cpf + "\n" +
  "Telefone: " + tw.local.dadosClienteBO.telefone + "\n\n" +
  "Dados do Contrato" + "\n" +
  "Agencia: " + tw.local.dadosContratoBO.agencia + "\n" +
  "Código: " + tw.local.dadosContratoBO.codigo + "\n";
}

Tarefa de Serviço [
Implementação {
Server > ECM_Server
Operação > Criar Documento
}

]

Variáveis [
Entrada {
  dadosClienteBO (dadosClienteBO)
  dadosContratoBO (dadosContratoBO)
}
Saída {
  pdfBase64 (string)
  formJSON (string)
}
Privado {
  stringPDF (string)
}

]]
```
 
**TratamentoFormularioJSON**
```Javascript
Tarefa de Script [
Script{
var formData = {
  cliente: tw.local.dadosClienteBO,
  contrato: tw.local.dadosContratoBO
};

tw.local.formJSON = JSON.stringify(formData);
}

Variáveis [
Entrada {
  dadosClienteBO (dadosClienteBO)
  dadosContratoBO (dadosContratoBO)
}
Saída {
  pdfBase64 (string)
  formJSON (string)
}
Privado {
  stringPDF (string)
}

]]
```

---

## Workflow

```Javascript
Variáveis [
  Entrada {
    meuFormulario (dadosClienteBO)
  }
  Saída { 
    formJSON (string)
    pdfB64 (string)
  }
  Privado {
    dadosClienteBO (dadosClienteBO)
    dadosContratoBO (dadosContratoBO)
  }
]
```

**Processo:**

*Formulário Digital (usuário)*
- Implementação:
  - ContratoPenhor
- Saída:
  – DadosCliente
  - DadoContrato

*TratamentoFormulário (sistema)*
- Implementação:
  - TratamentoFormulário
- Entrada:
  - DadosContratoBO
  - DadosClienteBO
- Saída:
  - pdfBase64
  - formJSON

*Salvando no Filenet PDF (sistema)*
- Implementação:
  - SaveFileNet
- Entrada:
  - pdfBase64
  - formJSON

*Salvando no Filenet JSON (sistema)*
- Implementação:
  - SaveFileNetJSON
- Entrada:
  - pdfBase64
  - formJSON

*Visualizando no Filenet (usuário)*
- Implementação:
  - ConexaoFilenet

## Coach
**ContratoPenhor**

```Javascript
Variáveis [
Entrada {
  dadosClientePenhor (dadosClienteBO)
  dadosContratoPenhor (dadosContratoBO)
}
Saída {
  dadosClientePenhor (dadosClienteBO)
  dadosContratoPenhor (dadosContratoBO)
}

Associação - View 1 {
dadosClientePenhor (dadosClienteBO)
}
Associação - View 2{
dadosContratoPenhor (dadosContratoBO)
}
]
```

**Views**

*DadosCliente:*
- Plain text:
  - nome (string)
  - cpf (string)
  - telefone (string)
 
```Javascript
Variáveis [
Dados de Negócio {
  dadosClienteBO (dadosClienteBO)
}

Associação [
plain text: nome - dadosClienteBO.nome
plain text: cpf - dadosClienteBO.cpf
plain text: telefone - dadosClienteBO.telefone
]

]
```

*DadosContrato:*
- Plain text:
  - agencia (string)
  - codigo (string)
 
```Javascript
Variáveis [
Dados de Negócio {
  dadosContratoBO (dadosContratoBO)
}

Associação [
plain text: agencia - dadosContratoBO.agencia
plain text: codigo - dadosContratoBO.codigo
]

]
```

---

ConexaoFilenet

Toolkit:
  - Case Manager UI
    - Content List

```Javascript
Configurações (Busque informações neste formato) [
Nome do Repositório: BAWTOS
ID de pasta: {97EA7AF0-0000-C1F4-B95E-B13B29DC44DE}
Serviço Graphql: https://<SEU PATH>/content-services-graphql/graphql
```

Para o ID da pasta, podemos acessar o Navigator (certifique-se de que esteja na mesma basta do Object Store utilizado) e clique em propriedades. O ID necessário é o segundo ou último apresentado: 
<img width="886" height="456" alt="image" src="https://github.com/user-attachments/assets/9f30b50c-25ba-4b8a-8704-549dc4116973" />
