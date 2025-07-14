# Sicoob - BAW and Filenet

## Acesso aos ambientes 

**Cloud pak:**  
https://cpd-cp4ba.apps.686bc0d26f7b07aa81b9e607.am1.techzone.ibm.com/zen/#/homepage

**Filenet Navigator – Ver documentos:**  
https://cpd-cp4ba.apps.686bc0d26f7b07aa81b9e607.am1.techzone.ibm.com/icn/navigator/

**Filenet Navigator – Configurações:**  
https://cpd-cp4ba.apps.686bc0d26f7b07aa81b9e607.am1.techzone.ibm.com/icn/navigator/?desktop=admin

**Filenet ACCE – Configurações:**  
https://cpd-cp4ba.apps.686bc0d26f7b07aa81b9e607.am1.techzone.ibm.com/cpe/acce/

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
- Host: `icp4adeploy-cmis-svc.cp4ba.svc.cluster.local`  
- Porta: `9443`  
- Caminho: `/cmis/openfncmis_wlp/services`  
- Servidor Seguro: `TRUE`  
- Repositório: `BAWTOS`  
- Usuário: `cpadmin`  
- Senha: `uPuks-JIugQ-zuljs-SufXD`


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
 
