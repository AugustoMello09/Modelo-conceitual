# Modelagem Conceitual

###  Estrutura de camadas do sistema

![mvc](https://user-images.githubusercontent.com/101072311/200664937-257765c1-4c76-4828-b0ee-fb01913388d5.png)


#### Anotações

- Na camada de controladores REST vai ficar nosso Resource.
- Por padrão na hora de trabalhar na criação das classes de entidade colocar no pacote domain.
- Service oferece operações e consulta para os controladores REST.
- A camada de serviço não tem contato com nenhuma tecnologia específicas(Não tem contato com o Banco, REST e com tela).
- A camada de serviço vai utilizar a camada de acesso a dados para realizar regras de negócio que por ventura ainda não foi implantado na camada domínio.
- A camada de acesso a dados(Repository) tem o papel de conversar com o banco de dados, é nela que vamos realizar as operações de salvar, alterar, excluir e consultar(Tudo que envolva SQL).
- Como não é uma boa prática colocar blocos de try catch no controladores REST, vamos fazer um Handler que é um objeto especial que vai interceptar e vai lançar a resposta Http correta.
- Em teste realizados, o uso de @JsonManagedReference/@JsonBackRefence apresentou alguns problemas com o
envio de dados Json em requisições .
Assim, ao invés de usar @JsonManagedReference/@JsonBackRefence, vamos simplesmente utilizar o
@JsonIgnore no lado da associação que não deve ser serializada.

### Criando a classe de controle REST
começando pela categoria.
~~~JAVA
@RestController
@RequestMapping(value = "/categorias")
public class CategoriaResource {


  @RequestMapping(method = RequestMothod.GET)
  public String listar(){
    return "REST está funcionando";
  }

}
~~~
#### Criando nossa entidade Categoria
 Criando nossa primeira entidade e colocando no pacote domain.
 Obs:Nela também acompanha Construtores, Getters, Setters e hashCode, mas não foi colocado na parte da documentação para não atrapalhar a visualização.
~~~JAVA
public class Categoria implements Serializable{
	private static final long serialVersionUID = 1L;

	private Integer id;
	private String nome;
}
~~~

#### Fazendo atualização na classe de controle REST.

Fazendo atualização no método de listar.

~~~JAVA
  @RequestMapping(method = RequestMothod.GET)
  public List<Categoria> listar(){
    Categoria cat1 = new Categoria(1, "Informática");
    Categoria cat2 = new Categoria(2,"Escritório");

    List<Categoria> lista = new ArrayList<>();
    lista.add(cat1);
    lista.add(cat2);

    return lista;
  }
~~~

- Testando no navegador e no Postman se está funcioando a requisição.

![nav](https://user-images.githubusercontent.com/101072311/200009942-467746d6-1aa5-4362-af0c-e99c9dcff392.png)

![postam](https://user-images.githubusercontent.com/101072311/200009971-5fd0dae5-03fc-470b-bee9-4e656ed3e7b6.png)

#### Gerando tabela  Categoria automaticamente no Banco de teste H2
colocando as Anotações na classe Categoria para gerar no Banco
~~~JAVA
@Entity
public class Categoria implements Serializable{
	private static final long serialVersionUID = 1L;

  @Id
  @GeneratedValue (strategy = GenerationType.IDENTITY)
  private Integer id;

  private String nome;
}  
~~~

![h2](https://user-images.githubusercontent.com/101072311/200090910-de65e94b-9e5d-4f87-9817-aa2ac8c6b2a0.png)

#### Criando a camada de acesso a dados(Repository)
Criando um Repository para acessar as categorias
~~~JAVA
@Repository
public interface CategoriaRepository extends JpaRepository<Categoria, Integer> {
}
~~~

#### Criando a camada Service
A camada Service vai servir de operações e consultas para Categoria e implantado um serviço que busca uma Categoria.
~~~JAVA
@Service
public class CategoriaService {

	@Autowired/*A dependência e automaticamente instanciada pelo Spring */
	private CategoriaRepository repo;

	public Categoria find(Integer id) {
		Optional<Categoria> obj = repo.findById(id);
		return obj.orElse(null);
	}

}
~~~
#### Atualizando os controladores REST
Atualizando para os controladores serem capazes de buscar uma Categoria.
~~~JAVA
@RestController
@RequestMapping(value = "/categorias")
public class CategoriaResource {

	@Autowired
	private CategoriaService service;

	@RequestMapping(value ="/{id}", method = RequestMethod.GET)
	public ResponseEntity<?> find(@PathVariable Integer id) {
		Categoria obj = service.find(id);
		return ResponseEntity.ok().body(obj);

	}
}
~~~
Incluindo manualmente no banco teste H2

~~~SQL
INSERT INTO CATEGORIA(NOME) VALUES ('Informática');
INSERT INTO CATEGORIA(NOME) VALUES ('Escritório');  

SELECT * FROM CATEGORIA
~~~

![dbh2](https://user-images.githubusercontent.com/101072311/200130932-1932dccc-675e-401b-bdb8-2bd8572da296.png)

- Testando a requisição no Postman.

/categorias/1

![cat1](https://user-images.githubusercontent.com/101072311/200131282-db0b348e-23c0-4695-9345-0c5360576505.png)

/categorias/2

![cat2](https://user-images.githubusercontent.com/101072311/200131281-8138f550-88a7-46fc-82c0-0f07956250c8.png)

#### Instanciando CategoriaRepository
para não ter o trabalho de instanciar manualmente os objetos  Categoria no banco de dados, vamos fazer isso automaticamente.
~~~JAVA
  @Autowired
	private CategoriaRepository categoriaRepository;

	public static void main(String[] args) {
		SpringApplication.run(EstudomcApplication.class, args);
	}

	@Override
	public void run(String... args) throws Exception {

		Categoria cat1 = new Categoria(null, "Informática");
		Categoria cat2 = new Categoria(null, "Escritório");

		categoriaRepository.saveAll(Arrays.asList(cat1, cat2));


	}
~~~
#### implementado a entidade Produto
Fazendo as associações entra elas e instanciar os objetos.
Obs:Nela também acompanha Construtores, Getters, Setters e hashCode, mas não foi colocado na parte da documentação para não atrapalhar a visualização.
~~~JAVA
@Entity
public class Produto implements Serializable {
	private static final long serialVersionUID = 1L;

	@Id
	@GeneratedValue(strategy =GenerationType.IDENTITY)
	private Integer id;

	private String nome;
	private Double preco;

	@ManyToMany
	@JoinTable(name = "PRODUTO_CATEGORIA",
	joinColumns = @JoinColumn(name= "produto_id"),
	inverseJoinColumns = @JoinColumn(name = "categoria_id"))
	private List<Categoria> categorias = new ArrayList<>();
}
~~~

##### Criando ProdutoRepository

~~~JAVA
@Repository
public interface ProdutoRepository extends JpaRepository<Produto, Integer> {
}
~~~

##### Instanciando os objetos e Repository na classe de execução.

~~~JAVA
  @Autowired
	private ProdutoRepository produtoRepository;

  Produto p1 = new Produto(null, "Computador", 2000.00);
	Produto p2 = new Produto(null, "Impressora", 800.00);
	Produto p3 = new Produto(null, "Mouse", 80.00);

	cat1.getProdutos().addAll(Arrays.asList(p1, p2, p3));
	cat2.getProdutos().addAll(Arrays.asList(p2));

	p1.getCategorias().addAll(Arrays.asList(cat1));
	p2.getCategorias().addAll(Arrays.asList(cat1, cat2));
	p3.getCategorias().addAll(Arrays.asList(cat1));

  categoriaRepository.saveAll(Arrays.asList(cat1, cat2));
  produtoRepository.saveAll(Arrays.asList(p1, p2, p3));
~~~

Testando no H2 o relacionamento

![select produtos](https://user-images.githubusercontent.com/101072311/200185181-79c68755-1f45-4b26-a216-a2b6f2e1321d.png)

![categoria](https://user-images.githubusercontent.com/101072311/200185187-fb07e001-b18e-4c1d-98eb-8437dc78b8c4.png)


#### Protegendo da referência cítrica no Json

~~~JAVA
@JsonManagedReference
	@ManyToMany(mappedBy = "categorias")
	private List<Produto> produtos  = new ArrayList<>();
~~~

~~~JAVA
@JsonBackReference
	@ManyToMany
	@JoinTable(name = "PRODUTO_CATEGORIA",
	joinColumns = @JoinColumn(name= "produto_id"),
	inverseJoinColumns = @JoinColumn(name = "categoria_id"))
	private List<Categoria> categorias = new ArrayList<>();
~~~

#### Corrigindo erro de busca vazia
Incluindo um tratamento de Exceções, agora o método de serviço ele lança uma exceção caso o Id não exista.  

![código que não existe](https://user-images.githubusercontent.com/101072311/200188736-8d520948-e24a-4fad-a516-121b7babf147.png)

~~~JAVA
public Categoria find(Integer id) {
		 Optional<Categoria> obj = repo.findById(id);
		return obj.orElseThrow(() -> new ObjectNotFoundException(
		 "Objeto não encontrado! Id: " + id + ", Tipo: " + Categoria.class.getName()));
		}
~~~

##### Criando um Hendler para lançar o erro
Como não é uma boa prática colocar blocos de try catch no controladores REST, vamos fazer um Handler que é um objeto especial que vai interceptar e vai lançar a resposta Http correta.

##### ResourceExceptionHandler

~~~JAVA
@ControllerAdvice
public class ResourceExceptionHandler  {

  @ExceptionHandler(ObjectNotFoundException.class)
  public ResponseEntity<StandardError> objectNotFound(ObjectNotFoundException e, HttpServletRequest request){

		StandardError err = new StandardError(HttpStatus.NOT_FOUND.value(), e.getMessage(), System.currentTimeMillis());
		return ResponseEntity.status(HttpStatus.NOT_FOUND).body(err);
	}
}
~~~

##### StandardError

~~~JAVA
public class StandardError implements Serializable {

	private static final long serialVersionUID = 1L;

	private Integer status;
	private String msg;
	private Long timeStamp;
}
~~~

![erro](https://user-images.githubusercontent.com/101072311/200189837-876186dd-d0c2-4036-ae8b-39a59d7bd645.png)

#### implementado a entidade Cidade
Com o relacionamento de muitos para um.
Obs:Nela também acompanha Construtores, Getters, Setters e hashCode, mas não foi colocado na parte da documentação para não atrapalhar a visualização.
~~~JAVA
@Entity
public class Cidade implements Serializable {
	private static final long serialVersionUID = 1L;

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;
	private String nome;

	@ManyToOne
	@JoinColumn(name ="estado_id")
	private Estado estado;

}
~~~

##### Criando CidadeRepository

~~~JAVA
@Repository
public interface CidadeRepository extends JpaRepository<Cidade, Integer> {
}
~~~

##### Instanciando os objetos e Repository na classe de execução.

~~~JAVA
@Autowired
private CidadeRepository cidadeRepository;

Cidade c1 = new Cidade(null, "Uberlândia", est1);
Cidade c2 = new Cidade(null, "São Paulo", est2);
Cidade c3 = new Cidade(null, "Campinas", est2);

cidadeRepository.saveAll(Arrays.asList(c1, c2, c3));
~~~

#### implementado a entidade Estado
Com o relacionamento de um para muitos.
Obs:Nela também acompanha Construtores, Getters, Setters e hashCode, mas não foi colocado na parte da documentação para não atrapalhar a visualização.
~~~JAVA
@Entity
public class Estado implements Serializable {
	private static final long serialVersionUID = 1L;

		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Integer id;

		private String nome;

		@OneToMany(mappedBy = "estado")
		private List<Cidade> cidade = new ArrayList<>();

}
~~~

##### Criando EstadoRepository

~~~JAVA
@Repository
public interface EstadoRepository extends JpaRepository<Estado, Integer> {
}
~~~

##### Instanciando os objetos e Repository na classe de execução.

~~~JAVA
@Autowired
private EstadoRepository estadoRepository;

Estado est1 = new Estado(null, "Minas Gerais");
Estado est2 = new Estado(null, "São Paulo");

// Adicionando cada cidade em seu estado
est1.getCidade().addAll(Arrays.asList(c1));
est2.getCidade().addAll(Arrays.asList(c2, c3));

estadoRepository.saveAll(Arrays.asList(est1, est2));
~~~
Testando no banco teste H2

![estado](https://user-images.githubusercontent.com/101072311/200338942-a7b48af5-7589-4e8e-853c-afa256bd9c21.png)

![cidade](https://user-images.githubusercontent.com/101072311/200338953-16b9973c-adbb-40c5-81bf-ee3e88807b40.png)


#### Criando um enumerado de TipoCliente

~~~JAVA
public enum TipoCliente {
	PESSOAFISICA(1, "Pessoa Física"),
	PESSOAJURIDICA(2, "Pessoa Jurídica");

	private int cod;
	private String descricao;

	private TipoCliente(int cod, String descricao) {
		this.cod = cod;
		this.descricao = descricao;
	}

	public int getCod() {
		return cod;
	}

	public String getDescricao() {
		return descricao;
	}

	public static TipoCliente toEnum(Integer cod) {

		if (cod == null) {
			return null;
		}

		for (TipoCliente x : TipoCliente.values()) {
			if (cod.equals(x.getCod())) {
				return x;
			}
		}

		throw new IllegalArgumentException("Id inválido: " + cod);

	}

}
~~~
#### implementado a entidade Cliente
Obs:Nela também acompanha Construtores, Getters, Setters e hashCode, mas não foi colocado na parte da documentação para não atrapalhar a visualização.
~~~JAVA
@Entity
public class Cliente implements Serializable {
	private static final long serialVersionUID = 1L;

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;

	private String nome;
	private String email;
	private String cpfOuCnpj;
	private Integer tipo;

	@OneToMany(mappedBy = "cliente")
	private List<Endereco> enderecos = new ArrayList<>();

	@ElementCollection
	@CollectionTable(name ="TELEFONE")
	private Set<String> telefones = new HashSet<>();
}
~~~

##### Criando ClienteRepository

~~~JAVA
@Repository
public interface ClienteRepository extends JpaRepository<Cliente, Integer> {
}
~~~

##### Instanciando os objetos e Repository na classe de execução.

~~~JAVA
@Autowired
private ClienteRepository clienteRepository;

Cliente cli1 = new Cliente(null, "Maria Silva", "maria@gmail.com", "36378912377", TipoCliente.PESSOAFISICA);

// Salvando os dois telefones
cli1.getTelefones().addAll(Arrays.asList("27363323", "93838393"));

//Salvando os endereços
cli1.getEnderecos().addAll(Arrays.asList(e1, e2));

clienteRepository.saveAll(Arrays.asList(cli1));
~~~

#### implementado a entidade Endereco
Obs:Nela também acompanha Construtores, Getters, Setters e hashCode, mas não foi colocado na parte da documentação para não atrapalhar a visualização.
~~~JAVA
@Entity
public class Endereco implements Serializable {
	private static final long serialVersionUID = 1L;

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;

	private String logradouro;
	private String numero;
	private String complemento;
	private String bairro;
	private String cep;

	@ManyToOne
	@JoinColumn (name ="cliente_id")
	private Cliente cliente;

	@ManyToOne
	@JoinColumn (name ="cidade_id")
	private Cidade cidade;

}
~~~

##### Criando ClienteRepository

~~~JAVA
@Repository
public interface EnderocoRepository extends JpaRepository<Endereco, Integer> {
}
~~~

##### Instanciando os objetos e Repository na classe de execução.

~~~JAVA
@Autowired
private EnderecoRepository enderecoRepository;

Endereco e1 = new Endereco(null, "Rua Flores", "360", "Apto 303", "Jardim", "38220834", cli1, c1);
Endereco e2 = new Endereco(null, "Avenida Matos", "105", "Sala 800", "Centro", "38777012", cli1, c2);

enderecoRepository.saveAll(Arrays.asList(e1, e2));
~~~

![maria](https://user-images.githubusercontent.com/101072311/200338921-69215553-4307-42ef-8611-4b7064715c28.png)

![teleclient1](https://user-images.githubusercontent.com/101072311/200338897-eea5a6d5-71a2-4ffa-870a-a387da85f7c0.png)

![endereco](https://user-images.githubusercontent.com/101072311/200338932-bb5c7fe7-2b6e-4a02-bafb-50ba7cb45cd8.png)

#### Criando o Endpoint Clientes/id

~~~JAVA
@RestController
@RequestMapping(value = "/clientes")
public class ClienteResource {

	@Autowired
	private ClienteService service;

	@RequestMapping(value ="/{id}", method = RequestMethod.GET)
	public ResponseEntity<?> find(@PathVariable Integer id) {
		Cliente obj = service.find(id);
		return ResponseEntity.ok().body(obj);

	}
}
~~~

#### Protegendo da referência cítrica no Json

Cliente
~~~JAVA
@JsonManagedReference
	@OneToMany(mappedBy = "cliente")
	private List<Endereco> enderecos = new ArrayList<>();
~~~
Endereço
~~~JAVA
@JsonBackReference
	@ManyToOne
	@JoinColumn (name ="cliente_id")
	private Cliente cliente;
~~~
Cidade
~~~JAVA
@JsonManagedReference
	@ManyToOne
	@JoinColumn(name ="estado_id")
	private Estado estado;
~~~
Estado
~~~JAVA
@JsonBackReference
		@OneToMany(mappedBy = "estado")
		private List<Cidade> cidade = new ArrayList<>();
~~~

#### Criando ClienteService

~~~JAVA
@Service
public class ClienteService {

	@Autowired
	private ClienteRepository repo;

	public Cliente find(Integer id) {
		 Optional<Cliente> obj = repo.findById(id);
		return obj.orElseThrow(() -> new ObjectNotFoundException(
		 "Objeto não encontrado! Id: " + id + ", Tipo: " + Cliente.class.getName()));
		}

}
~~~

Testando o Endpoint no Postman (clientes/{id})

![testeEndpoint](https://user-images.githubusercontent.com/101072311/200357689-a0c8c2a7-08b7-430d-b2cf-606e5f0035ae.png)

![terminotest1](https://user-images.githubusercontent.com/101072311/200357704-b7b0d860-dac2-436b-bf61-0ad53b5854d7.png)

#### Criando a entidade Pedidos
Obs:Nela também acompanha Construtores, Getters, Setters e hashCode, mas não foi colocado na parte da documentação para não atrapalhar a visualização.
~~~JAVA
@Entity
public class Pedido implements Serializable {
	private static final long serialVersionUID = 1L;

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;
	private Date instante;

	@OneToOne(cascade=CascadeType.ALL, mappedBy ="pedido")
	private Pagamento pagamento;

	@ManyToOne
	@JoinColumn(name="cliente_id")
	private Cliente cliente;

	@ManyToOne
	@JoinColumn(name="endereco_de_entrega_id")
	private Endereco enderecoDeEntrega;
}  
~~~

#### Criando a classe Pagamento

~~~JAVA
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Pagamento implements Serializable {
	private static final long serialVersionUID = 1L;

	@Id
	private Integer id;
	private Integer estado;

	@OneToOne
	@JoinColumn(name ="pedido_id")
	@MapsId
	private Pedido pedido;
}  
~~~
#### Criando um enumerado de EstadoPagamento

~~~JAVA
public enum EstadoPagamento {
	PENDENTE(1, "Pendente"),
	QUITADO(2, "Quitado"),
	CANCELADO(3, "Cancelado");

	private int cod;
	private String descricao;

	private EstadoPagamento(int cod, String descricao) {
		this.cod = cod;
		this.descricao = descricao;
	}

	public int getCod() {
		return cod;
	}

	public String getDescricao() {
		return descricao;
	}

	public static EstadoPagamento toEnum(Integer cod) {

		if (cod == null) {
			return null;
		}

		for (EstadoPagamento x : EstadoPagamento.values()) {
			if (cod.equals(x.getCod())) {
				return x;
			}
		}

		throw new IllegalArgumentException("Id inválido: " + cod);

	}

}
~~~

#### Criando as SubClasses de pagamento

##### Pagamento com Boleto
~~~JAVA
@Entity
public class PagamentoComBoleto  extends Pagamento{

	private static final long serialVersionUID = 1L;
	private Date dataVencimento;
	private Date dataPagamento;
}
~~~
##### Pagamento com Cartão
~~~JAVA
@Entity
public class PagamentoComCartao  extends Pagamento {

	private static final long serialVersionUID = 1L;

	private Integer numeroDeParcelas;
}
~~~

#### mapeamento OneToOne nos Pedidos
~~~JAVA
@OneToOne
	@JoinColumn(name ="pedido_id")
	@MapsId
	private Pedido pedido;

  @OneToOne(cascade=CascadeType.ALL, mappedBy ="pedido")
	private Pagamento pagamento;
~~~

##### Criando PagamentoRepository

~~~JAVA
@Repository
public interface PagamentoRepository extends JpaRepository<Pagamento, Integer> {		
}
~~~

##### Criando PedidoRepository

~~~JAVA
@Repository
public interface PedidoRepository extends JpaRepository<Pedido, Integer> {		
}
~~~

##### Instanciando os objetos e Repository na classe de execução.

~~~JAVA
@Autowired
private PagamentoRepository pagamentoRepository;

@Autowired
private PedidoRepository pedidoRepository;

SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy HH:mm");

Pedido ped1 = new Pedido(null, sdf.parse("30/09/2017 10:32"), cli1, e1);
Pedido ped2 = new Pedido(null, sdf.parse("10/10/2017 19:35"), cli1, e2);

Pagamento pagto1 = new PagamentoComCartao(null, EstadoPagamento.QUITADO, ped1, 6);
ped1.setPagamento(pagto1);

Pagamento pagto2 = new PagamentoComBoleto(null, EstadoPagamento.PENDENTE, ped2, sdf.parse("20/10/2017 00:00"),null);
ped2.setPagamento(pagto2);

cli1.getPedidos().addAll(Arrays.asList(ped1, ped2));

pedidoRepository.saveAll(Arrays.asList(ped1, ped2));
pagamentoRepository.saveAll(Arrays.asList(pagto1, pagto2));
~~~

Testando no bando de dados H2

![pagamento](https://user-images.githubusercontent.com/101072311/200578236-5bb1a032-9003-4660-ac8b-401713d4e2be.png)

![pagamentoboleto](https://user-images.githubusercontent.com/101072311/200578251-0d75b6f8-b35c-47eb-9148-4a7550327ee7.png)

![pagamentoCartão](https://user-images.githubusercontent.com/101072311/200578263-ce479eaa-9df7-477e-be7f-cb04478de2ec.png)

![pedido](https://user-images.githubusercontent.com/101072311/200578271-367c4604-7792-4058-9535-90a49a2fcc0c.png)


#### Entidade ItemPedido
Obs:Nela também acompanha Construtores, Getters, Setters e hashCode, mas não foi colocado na parte da documentação para não atrapalhar a visualização.
~~~JAVA
@Entity
public class ItemPedido implements Serializable {
	private static final long serialVersionUID = 1L;

	@EmbeddedId
	private ItemPedidoPK id = new ItemPedidoPK();

	private Double desconto;
	private Integer quantida;
	private Double preco;
}  
~~~

##### Classe ItemPedidoPK
Obs:Nela também acompanha Construtores, Getters, Setters e hashCode, mas não foi colocado na parte da documentação para não atrapalhar a visualização.
~~~JAVA
@Embeddable
public class ItemPedidoPK implements Serializable {

	private static final long serialVersionUID = 1L;

	@ManyToOne
	@JoinColumn(name="pedido_id")
	private Pedido pedido;

	@ManyToOne
	@JoinColumn(name ="produto_id")
	private Produto produto;
}
~~~

##### Criando ItemPedidoRepository

~~~JAVA
@Repository
public interface PedidoRepository extends JpaRepository<ItemPedido, Integer> {		
}
~~~

##### Instanciando os objetos e Repository na classe de execução.

~~~JAVA
@Autowired
private ItemPedidoRepository itemPedidoRepository;

ItemPedido ip1 = new ItemPedido(ped1, p1, 0.00, 1, 2000.00);
ItemPedido ip2 = new ItemPedido(ped1, p3, 0.00, 2, 80.00);
ItemPedido ip3 = new ItemPedido(ped2, p2, 100.00, 1, 800.00);

ped1.getItens().addAll(Arrays.asList(ip1, ip2));
ped2.getItens().addAll(Arrays.asList(ip3));

p1.getItens().addAll(Arrays.asList(ip1));
p2.getItens().addAll(Arrays.asList(ip3));
p3.getItens().addAll(Arrays.asList(ip2));

itemPedidoRepository.saveAll(Arrays.asList(ip1, ip2, ip3));
~~~

Testando no bando de dados H2

![itempedido](https://user-images.githubusercontent.com/101072311/200649418-e985597e-2cb0-4553-985c-c5eac39be3e3.png)

#### Criando o Endpoint pedidos/id

~~~JAVA
@RestController
@RequestMapping(value = "/pedidos")
public class PedidoResource {

	@Autowired
	private PedidoService service;

	@RequestMapping(value ="/{id}", method = RequestMethod.GET)
	public ResponseEntity<?> find(@PathVariable Integer id) {
		Pedido obj = service.find(id);
		return ResponseEntity.ok().body(obj);

	}
}
~~~
#### Criando PedidoService

~~~JAVA
@Service
public class PedidoService {

	@Autowired
	private PedidoRepository repo;

	public Pedido find(Integer id) {
		 Optional<Pedido> obj = repo.findById(id);
		return obj.orElseThrow(() -> new ObjectNotFoundException(
		 "Objeto não encontrado! Id: " + id + ", Tipo: " + Pedido.class.getName()));
		}

}
~~~

Testando o Endpoint no Postman (pedidos/{id})


![pedidos1](https://user-images.githubusercontent.com/101072311/200660612-f423bd48-7cc9-4a41-adfd-7a372a7ddadf.png)

![pedidos2](https://user-images.githubusercontent.com/101072311/200660618-23a6b3da-a574-4a3a-b281-0d887480dd6f.png)

![pediso3](https://user-images.githubusercontent.com/101072311/200660625-6727079b-a21b-4205-bebd-10619bbad8bd.png)

![pedidos4](https://user-images.githubusercontent.com/101072311/200660630-69bbd2f1-9239-4082-b785-6550aef8ef2b.png)

### Atualizacao: utilizando somente JsonIgnore
Em teste realizados, o uso de @JsonManagedReference/@JsonBackRefence apresentou alguns problemas com o
envio de dados Json em requisições .
Assim, ao invés de usar @JsonManagedReference/@JsonBackRefence, vamos simplesmente utilizar o
@JsonIgnore no lado da associação que não deve ser serializada. Para isto faça:

 Para cada classe de domínio:
 Apague as anotações @JsonManagedReference existentes
 Troque as anotações @JsonBackRefence por @JsonIgnore

## Ferramentas e Tecnologias usadas nesse repositório 🌐
<div style="display: inline_block"><br>

<img align="center" alt="Augusto-Java" height="60" width="60" src=https://github.com/devicons/devicon/blob/master/icons/java/java-original.svg >
<img align="center" alt="Augusto-SpringBoot" height="60" width="60" src="https://raw.githubusercontent.com/devicons/devicon/1119b9f84c0290e0f0b38982099a2bd027a48bf1/icons/spring/spring-original-wordmark.svg">
<img align="center" alt="Augusto-SpringBoot" height="60" width="60" src="https://user-images.githubusercontent.com/101072311/200666111-2e4878bb-7d5c-4103-a159-fd00d0855a5d.png">

</div>    

## Teste o projeto 👁‍🗨

Download do projeto para testar em sua máquina: xxx

## Entre em contado 👋
  
<div>
  
  <a href = "joseaugusto.mello01@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white" target="_blank"></a>
  <a href="https://www.linkedin.com/in/jos%C3%A9-augusto-794a94234/" target="_blank"><img src="https://img.shields.io/badge/-LinkedIn-%230077B5?style=for-the-badge&logo=linkedin&logoColor=white" target="_blank"></a>   

  </div>
