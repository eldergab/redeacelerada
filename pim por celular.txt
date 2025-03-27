<?php
// Conexão com o banco de dados (ajuste conforme sua configuração)
$servername = "localhost";
$username = "root";
$password = "";
$dbname = "redeace";

$conn = new mysqli($servername, $username, $password, $dbname);

// Verifica se a conexão foi bem-sucedida
if ($conn->connect_error) {
    die("Conexão falhou: " . $conn->connect_error);
}

// Recebe os dados do formulário
if (!isset($_POST['pin'])) {
    $nome = $_POST['nome'];
    $email = $_POST['email'];
    $telefone = $_POST['telefone'];
    $chave_pix = $_POST['chave_pix'];
    $chave_link_convidante = $_POST['chave_link_convidante']; // Opcional

    // Gera um PIN aleatório e envia por SMS (implementar usando uma API de SMS)
    $pin = rand(1000, 9999);
    enviar_sms($telefone, $pin);

    // Armazena o PIN na sessão
    session_start();
    $_SESSION['pin'] = $pin;

    // Exibe a página de confirmação do PIN
    ?>
    <form action="processa_cadastro.php" method="post">
        <label for="pin">PIN:</label>
        <input type="text" id="pin" name="pin" required>
        <br><br>
        <input type="submit" value="Confirmar">
    </form>
    <?php
} else {
    // Verifica se o PIN foi informado
    $pin_informado = $_POST['pin'];

    // Verifica se o PIN é válido
    if ($pin_informado == $_SESSION['pin']) {
        // Gera uma chave link única
        $chave_link = gerar_chave_link();

        // Insere os dados no banco de dados
        $sql = "INSERT INTO usuarios (nome, email, telefone, chave_pix, chave_link, chave_link_pai) 
                VALUES ('$nome', '$email', '$telefone', '$chave_pix', '$chave_link', '$chave_link_convidante')";

        if ($conn->query($sql) === TRUE) {
            echo "Novo registro criado com sucesso";
        } else {
            echo "Erro: " . $sql . "<br>" . $conn->error;
        }

        // Redireciona para a página de confirmação
        header("Location: confirmacao.php?chave_link=$chave_link");
    } else {
        echo "PIN inválido";
    }
}

// Funções auxiliares (exemplos)
function enviar_sms($telefone, $pin) {
    // Implemente a lógica para enviar o SMS usando uma API de SMS
}

function gerar_chave_link() {
    // Gera uma string aleatória única para a chave link
    return bin2hex(random_bytes(16));
}

$conn->close();
?>