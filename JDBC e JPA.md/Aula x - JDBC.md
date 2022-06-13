# Aula 1

Para iniciar o programa, nós  devemos, primeiro

https://mvnrepository.com/artifact/mysql/mysql-connector-java







Consultas com JDBC.
Existem 3 interfaces para montar comandos SQL.

Statement - Executar o SQL comum.
PreparedStament - Exetuca o SQL parametizável.
CallableStatement - Executa stored procedures.

A mais usada é o PreparedStament.

Existem 3 métodos paa executar comandos SQL:
execute - Pode executar qualquer tipo SQL.
executeQuery - Usado para executar "SELECT".
executeUpdate - Usado para comandos de alteração de banco de dados (INSERT, UPDATE, DELETE, CREATE, ALTER).

Os mais executados são os dois últimos.

Consulta com JDBC
ResultSet - objeto que contem os dados de uma determinada consulta no banco de dados (normalmente com SELECT).

São utilizados os métodos getters para buscar dados do ResultSet. Tais como: getInt, getFloat e getString.

O métodos next() é utilizado para percorrer os registros do ResultSet. (Normalmente utilizado junto com while).


Criando um db.properties.
Serve para colocarmos o nome do usuário, senha e etc em um local separado. Nós podemos criar na raíz do projeto, ou seja, na mesma área que a pasta "scr" está. 

Criamos uma pasta e, dentro dela, adicionamos o arquivo "db.properties". Lá colocamos o nome do banco, a url do banco, usuário, senha e etc. Ou seja, tudo que for necessário para fazer a conecção no banco de dados.

A forma de implementar alguma propriedade é como em uma variável.
```
user=wallinson
password=senha
dburl=jdbc:mysql:localhost/nome_do_banco
...
```

Classe para pegar as propriedades
Após isso, criamos uma classe para fazer o acesso a essas propriedades.
```
public class DB{
    ...
}
```

Dentro dela, nós iremos importar a classe Properties e o FileInputStream para usarmos ela. Esta últica deverá estar dentro de um try cacth tratando do IOException. Afinal de conta é ela que irá procurar o arquivo.
```
import java.io.FileInputStream;
import java.util.Properties;

public class DB{

    private static Properties loadProperties(){
        try(FileInputStream fs = new FileInputStream("db.properties")){
            Properties props = new Properties();
            props.load(fs);
            return props;
        } catch (IOException e){
            System.out.println("Erro ao buscar arquivo de conexão");
            e.getMessage();
        }
    }
}
```

Além disso, nós precisamos criar um método para fazer essa conexão. E, nessa conexão, devemos tratar a exception que vier, no caso a SQL.
```
private static Connection conn = null;

public static Connection getConnection(){
    try{
        if (conn == null){
            Properties props = loadProperties();
            String url = props.getProperty("propiredade da url");
            conn = DriveManager.getConnection(url, props);
        }
    System.out.println("Conectado com sucesso!");
    return conn;
    } cacth(SQLException e){
        System.out.println("Erro ao se conectar!");
        e.getMessage();
    }
}
```

E, para finalizar, devemos ter métodos que irão fechar a conexão. Por precisarem de tratamento, eles devem passar por aqui, primeiro.
```
public static void closeConnection(){
    if(conn == null){
        try{
            conn.close();
        } catch(SQLException e){
            System.out.println("Erro ao fechar conexão!");
        }     
    }
}

public static void closeStatement(Statement st){
    if(st != null){
        try{
            st.close();
        } catch (SQLException e){
            e.printStackTrace();
        }
    }
}

public static void closeResultSet(ResultSet rs){
    if(rs != null){
        try{
            rs.close();
        } catch (SQLException e){
            e.printStackTrace();
        }
    }
}
```

Agora nós podemos fazer essas chamadas na nossa classe principal ao importar a classe que foi criada para fazer a conexão.
```
public static void main(String[] args){
    Connection conn = DB.getConnection;
    Db.closeConnection();
}
```




// Recuperar dados.
Interface - Statement.createStatement().
Método - executeQuery()

Implementação:
```
// Variáveis para conexão e Recuperar dados
Connection conn = null;
Statement st = null;
ResultSet rs = null;

try{
    // Conexão
    conn = DB.getConnection();
    // Interface
    st = conn.createStatement();

    // Método para recuperar dados
    rs = st.executeQuery("SELECT * FROM table");

    // next() para pular para o próximo dado até finalizar. Quando finaliza, ele retorna false.
    while(rs.next()){
        // Os getters para buscar as chaves
        // getInt e getString
        System.out.println(rs.getInt("id") +", " + rs.getString("nome"));
    }
} catch (SQLException e){
    e.printStackTrace();
}

// É sempre importante fechar a conexão para evitar vazamento de memória.
finally{
    DB.closeResultSet(rs);
    DB.closeStatement(st);
    DB.closeConnection();
}
```



// Inserir dados
Interface - PreparedStatement.prepareStatement()
Método - executeUpdate()

Getter
getGeneratedKeys().getInt()

Setters
- setString()
- setInt()
- setDouble()
> Há diversos outros. Esses são os que foram usados no código abaixo.

SetDate
Para utilizar o setDate, é muito importante criar um objeto SimpleDateFormat e, em seguida, colocar dessa forma:
```
setDate(x, new java.sql.Date(sdf.parse("<data>").getTime()));
```

Parâmtro dos Setter
Dentro dos parenteses nós especifivamos qual será a interrogação que será modificada e, separada por vírgula, o seu valor. No código abaixo veremos o exemplo.

Implementação:
```
// Variáveis para conexão e Recuperar dados
Connection conn = null;
PreparedStatement st = null;
SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy");

try{
    // Conexão
    conn = DB.getConnection();
    // Interface
    st = conn.prepareStatement(
        "INSERT INTO seller " +
        "(Name, Email, BithDate, BaseSalary, DepartmentId) " +
        "VALUES "+
        "(?, ?, ?, ?, ?)",
        Statement.RETURN_GENERATED_KEYS // Para ser captado pelo ResultSet.
    );

    //Implementação dos dados
    st.setString(1, "Carl Purple"); //Representa, na ordem, a interrogação e a string.
    st.setString(2, "email@gmail.com");
    st.setDate(3, new java.sql.Date(sdf.parse("22/04/1985").getTime()));
    st.setDouble(4, 3000.0);
    st.setInt(5, 4);

    //Método
    int rowsAffected = st.executeUpdate();
    
    if(rowsAffected > 0){
        ResultSet rs = st.getGeneratedKeys();
        while (rs.next()){
            int id = rs.getInt(1);
            System.out.println("Id = "+ id);
        }
    } else{
        System.out.println("Nenhuma linha foi alterada");
    }

} catch (SQLException e){
    e.printStackTrace();
}

// É sempre importante fechar a conexão para evitar vazamento de memória.
finally{
    DB.closeStatement(st);
    DB.closeConnection();
}
```




//Atualizando dados
Interface - PreparedStatement.prepareStatement()
Método - executeUpdate()

Getter
getGeneratedKeys().getInt()

Setters
- setString()
- setInt()
- setDouble()
> Há diversos outros. Esses são os que foram usados no código abaixo.

Implementação:
```
// Variáveis para conexão e Recuperar dados
Connection conn = null;
PreparedStatement st = null;

try{
    // Conexão
    conn = DB.getConnection();
    // Interface
    st = conn.prepareStatement(
        "UPDATE seller " +
        "SET BaseSalary = BaseSalary + ? " +
        "WHERE " +
        "(DepartamentId = ?)"
    );

    //Atualização dos dados.
    st.setDouble(1, 200.0);
    st.setInt(2, 2);

    //Método
    int rowsAffected = st.executeUpdate();
    
    if(rowsAffected > 0){
        ResultSet rs = st.getGeneratedKeys();
        while (rs.next()){
            int id = rs.getInt(1);
            System.out.println("Id = "+ id);
        }
    } else{
        System.out.println("Nenhuma linha foi alterada");
    }

} catch (SQLException e){
    e.printStackTrace();
}

// É sempre importante fechar a conexão para evitar vazamento de memória.
finally{
    DB.closeStatement(st);
    DB.closeConnection();
}
```





// Deletando dados
Interface - PreparedStatement.prepareStatement()
Método - executeUpdate()

Getter
getGeneratedKeys().getInt()

Setters
- setString()
- setInt()
- setDouble()
> Há diversos outros. Esses são os que foram usados no código abaixo.

Implementação:
```
// Variáveis para conexão e Recuperar dados
Connection conn = null;
PreparedStatement st = null;

try{
    // Conexão
    conn = DB.getConnection();
    // Interface
    st = conn.prepareStatement(
        "DELETE FROM department " +
        "WHERE " +
        "Id = ?"
    );

    //Deletando dados.
    st.setInt(1, 2);

    //Método
    int rowsAffected = st.executeUpdate();
    
    if(rowsAffected > 0){
        ResultSet rs = st.getGeneratedKeys();
        while (rs.next()){
            int id = rs.getInt(1);
            System.out.println("Id = "+ id);
        }
    } else{
        System.out.println("Nenhuma linha foi alterada");
    }

} catch (SQLException e){
    e.printStackTrace();
}

// É sempre importante fechar a conexão para evitar vazamento de memória.
finally{
    DB.closeStatement(st);
    DB.closeConnection();
}
```