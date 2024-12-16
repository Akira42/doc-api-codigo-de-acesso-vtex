# doc-api-codigo-de-acesso-vtex
Neste repositório, tentarei explicar como podemos implementar o fluxo de envio e recebimento do código da VTEX para login ou criar uma nova conta/senha num componente de login totalmente customizado

## 1 - Solicitar o código
 Para solicitar o código de acesso de 6 digitos da VTEX, para fazer login ou criar uma nova conta ou senha como usuário do site, precisamos criar duas requisições conforme abaixo:

### 1.1 - startLogin
Sinceramente não consegui entender ainda oque essa requisição faz exatamente (suponho que crie um cookie de autenticação), mas aparentemente ela é necessária antes de fazer a requisição que de fato solicita o código.
 Endpoint: /api/vtexid/pub/authentication/startlogin
 Method: POST
 Headers: { "Content-Type": "multipart/form-data" }
 Data: ex: { 
   accountName : "persol", (Nome do vendor)
   scope : "persol", (Nome do vendor)
   returnUrl: "/account",
   callbackUrl: "/api/vtexid/oauth/finish?popup=false",
   user : "email@email.com" (e-mail que receberá o código)
 }
__IMPORTANTE!: O Objeto Data, deve ser convertido em um objeto FormData antes de ser enviado, leia o passo 3__

Assim concluida, ela deve chamar a requisição do passo 1.2, deve parecer algo como:

```
   const startLogin = () => { 
      const loginBodyData = {
        accountName : "persol",
        scope: "persol",
        returnUrl : "/account",
        callbackUrl : "/api/vtexid/oauth/finish?popup=false",
        user : inputEmail,
        fingerprint: ""
      }

      axios({
        method: "post",
        url: `/api/vtexid/pub/authentication/startlogin`,
        data: formatFormData(loginBodyData),
        headers: { "Content-Type": "multipart/form-data" },
      }).then(function (response) {
        sendCode();
        console.log(response);
      }).catch(function (response) {
        console.log(response);
      });
    };
```

### 1.2 - accessKey
Essa requisição é responsável por enviar o código de seis digitos para o email informado.
 
 Endpoint: /api/vtexid/pub/authentication/accesskey/send?deliveryMethod=email
 Method: POST
 Headers: { "Content-Type": "multipart/form-data" }
 Data: ex: { 
   email : "email@email.com",
   locale : "pt-BR", 
   recaptcha : "",
   recaptchaToken : "",
   parentAppId : "vtex.login@2.66.0"
 }

__IMPORTANTE!: O Objeto Data, deve ser convertido em um objeto FormData antes de ser enviado, leia o passo 3__
 OBS: Pelo que validei, no objeto Data, apenas o campo de email é obrigatório, porém creio que seja uma boa prática enviar o parentAppId.
 OBS 2: O campo recaptchaToken, é necessário apenas quando o usuário está com login bloqueado temporariamente, por muitas tentativas de login, por exemplo, é preciso entender como e onde conseguimos esse token e implementar nesse campo.

Sua requisição deve ficar parecida com isso:
```
    const sendCode = () => { 
      const requestBodyData = {
        email : inputEmail
      }

      axios({
        method: "post",
        url: "/api/vtexid/pub/authentication/accesskey/send?deliveryMethod=email",
        data: formatFormData(requestBodyData),
        headers: { "Content-Type": "multipart/form-data" },
      }).then(function (response) {
        console.log(response);
      }).catch(function (response) {
        console.log(response);
      });
    };

```

## 2 - Validando o código de acesso
Para validar o código de acesso recebido no e-mail e cadastrar uma nova senha, basta fazer a requisição abaixo:

 Endpoint: /api/vtexid/pub/authentication/classic/setpassword?expireSessions=true
 Method: POST
 Headers: { "Content-Type": "multipart/form-data" }
 Data: ex: { 
    url : "/api/vtexid/pub/authentication/classic/setpassword?expireSessions=true",
    login : "teste@teste.com",
    accesskey: "123456", 
    password:'@P4ssw0rd'
 }

 O campo de login, deve ser o email que recebeu o código, accessKey é o código de 6 digitos, e a senha, bem... é a senha

 __IMPORTANTE!: a senha deve ser validada no front, pois aparentemente a VTEX não tem uma validação server side, e aceita qualquer coisa.__
 __IMPORTANTE!: O Objeto Data, deve ser convertido em um objeto FormData antes de ser enviado, leia o passo 3__

sua requisição deve se parecer com isso:

```
  const handleCodeVerification = async (input) => {
    const inputEmail = "vtex@seriedesign.com.br"; //deixar dinâmico
  
    // faz a requisição para enviar o código de acesso VTEX
    const validateCode = () => { 
      const requestBodyData = {
        url : "/api/vtexid/pub/authentication/classic/setpassword?expireSessions=true",
        login : inputEmail, 
        accesskey: input.accessCode,
        password:'@teste'
      }
  
      axios({
        method: "post",
        url: "/api/vtexid/pub/authentication/accesskey/send?deliveryMethod=email",
        data: formatFormData(requestBodyData),
        headers: { "Content-Type": "multipart/form-data" },
      }).then(function (response) {
        console.log(response);
      }).catch(function (response) {
        console.log(response);
      });
    };

  }
```


### 3 - Corvetendo em FormData
O código abaixo recebe um objeto com os dados de cada formulário e formata no objeto FormData, provavelmente pode ser melhorado, mas ele já faz oque é preciso para formatar os dados para cada endpoint.
```
  const formatFormData = (data) => {
    let bodyFormData = new FormData();

    Object.entries(data).forEach(([dataItemKey, dataItemVal] ) => {
      bodyFormData.append(dataItemKey, dataItemVal);
    });

    return bodyFormData;
  }
```

 

 
 
