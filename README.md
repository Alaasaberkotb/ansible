stage('Setup Authentication') {
    steps {
        script {
            def authScript = """\
                function authenticate(helper, paramsValues, credentials) {
                    var requestUri = new URI("\${paramsValues.get('loginUrl')}", false);
                    var requestMethod = "POST";
                    var requestBody = "username=" + credentials.get('username') + "&password=" + credentials.get('password');

                    var requestMessage = helper.prepareMessage();
                    requestMessage.getRequestHeader().setURI(requestUri);
                    requestMessage.getRequestHeader().setMethod(requestMethod);
                    requestMessage.setRequestBody(requestBody);
                    helper.sendAndReceive(requestMessage);

                    var responseMessage = requestMessage.getResponseBody().toString();
                    if (responseMessage.includes("Login Successful")) {
                        return true;
                    } else {
                        return false;
                    }
                }
            """.stripIndent()

            writeFile file: 'auth-script.js', text: authScript

            // Debugging step: Check if the file is correctly written
            bat 'type auth-script.js'

            // Copy script into OWASP ZAP container
            bat 'docker cp auth-script.js owasp:/zap/scripts/'
        }
    }
}