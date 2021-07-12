# SK Happy Delivery

## 1. 분석/설계

### 이벤트스토밍

- url: http://www.msaez.io/#/storming/wf1WRjEyVVWd1Abldu2nsM6FwbL2/58c36eee763868e2a4b6cc1f019683fe


### 이벤트 도출

![이벤트도출결과](https://user-images.githubusercontent.com/45377807/125230473-ca215d00-e313-11eb-8866-2bdbfd5480be.png)




### 액터, 커맨드 부착
![액터 커맨드 부착](https://user-images.githubusercontent.com/45377807/125232032-c3e0b000-e316-11eb-8af4-cf98f7b97dac.png)
 

### 어그리게잇 묶기
![어그리게잇으로 묶기](https://user-images.githubusercontent.com/45377807/125232099-e7a3f600-e316-11eb-97ae-275b528205bd.png)


### 바운디드 컨텍스트 묶기
<img width="830" alt="바운디드컨텍스트 묶기" src="https://user-images.githubusercontent.com/45377807/125232137-f7bbd580-e316-11eb-82f0-293cd566faac.png">


### 폴리시 부착
<img width="828" alt="폴리시 추가" src="https://user-images.githubusercontent.com/45377807/125232200-16ba6780-e317-11eb-9e1d-db6cce330b92.png">


### 완성된 모형(실선은 Req/Res, 점선은 Pub/Sub)
<img width="830" alt="이벤트스토밍 결과" src="https://user-images.githubusercontent.com/45377807/125232289-423d5200-e317-11eb-83e9-f936ad5ea2c4.png">



### 헥사고날 아키텍처 다이어그램 도출
![헥사고날 아키텍쳐 다이어그램](https://user-images.githubusercontent.com/45377807/125297244-08456d80-e362-11eb-9164-e53cbfa52901.png)




## 2. 구현

분석/설계단계에서 도출된 헥사고날 아키넥처에 따라, 각 바운디트 컨텍스트 별로 대변되는 마이크로 서비스들을 스푸링부트로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다

![메이븐 실행](https://user-images.githubusercontent.com/45377807/125234914-96970080-e31c-11eb-933b-7008c23038bf.png)


### Domain Driven Design의 적용
- 각 서비스 내에 도출된 핵심 어그리게잇 객체를 엔티티로 선언했다. 이때 가능한 현업에서 사용하는 유비쿼터스 랭귀지를 사용하려 노력했다.


      package skhappydelivery;
	
	  import javax.persistence.Entity;
	  import javax.persistence.GeneratedValue;
	  import javax.persistence.GenerationType;
	  import javax.persistence.Id;
	  import javax.persistence.PostPersist;
	  import javax.persistence.PostUpdate;
	  import javax.persistence.Table;
	  
	  import org.springframework.beans.BeanUtils;
	  
	  @Entity
	  @Table(name="Order_table")
	  public class Order {
	
    	@Id
    	@GeneratedValue(strategy=GenerationType.AUTO)
    	private Long orderId;
    	private Long customerId;
    	private String customerName;
    	private String customerAddress;
    	private Integer phoneNumber;
    	private Long menuId;
    	private Integer menuCount;
    	private Integer menuPrice;
    	private Long storeId;
    	private String orderStatus;  
	
    	
    	@PostPersist
    	public void onPostPersist(){
	
   	     skhappydelivery.external.Payed Payed = new skhappydelivery.external.Payed();
   	     // mappings goes here

 	       Payed.setCustomerId(this.customerId);
 	       Payed.setOrderId(this.orderId);
  	      Payed.setStoreId(this.storeId);
  	      Payed.setTotalPrice(this.menuCount * this.menuPrice);

 	       OrderApplication.applicationContext.getBean(skhappydelivery.external.PayService.class)
 	           .payed(Payed);
 	   }


	  @PostUpdate
 	  public void onPostUpdate(){
  	     OrderCanceled orderCanceled = new OrderCanceled();
	
		        //Reject >>> publish
				if(this.orderStatus=="orderCanceled"){

					BeanUtils.copyProperties(this, orderCanceled);

					orderCanceled.setOrderStatus(this.orderStatus);
			
					System.out.println(" PUBLISH orderCanceledOBJ:  " +orderCanceled.toString());
				
					orderCanceled.publishAfterCommit();
		
				}
	
    			}
	
		}

### 주문 생성
![오더 증적1](https://user-images.githubusercontent.com/45377807/125314385-1bac0500-e371-11eb-829e-feb8a4158772.png)
![오더 증적2](https://user-images.githubusercontent.com/45377807/125314416-21094f80-e371-11eb-9243-44502ac1928b.png)
### 오더에 따른 결제 호출(Req/Res)
![결제 증적1](https://user-images.githubusercontent.com/45377807/125314434-26669a00-e371-11eb-8719-caa3e35fd054.png)
### 결제 후 스토어에서 주문접수
![스토어오더접수 증적](https://user-images.githubusercontent.com/45377807/125314603-4eee9400-e371-11eb-9aa3-e3484943e402.png)
### 고객이 주문취소(주문취소에 따른 스토어의 주문접수 취소)
#### 오더 정상생성 확인
![오더취소 증적1](https://user-images.githubusercontent.com/45377807/125314848-865d4080-e371-11eb-97e5-3b713334fef2.png)
![오더취소 증적2](https://user-images.githubusercontent.com/45377807/125314854-88270400-e371-11eb-90f5-ab4e83a581f2.png)
#### 생성된 주문이 스토어에서 주문 접수됨을 확인
![오더취소 증적3](https://user-images.githubusercontent.com/45377807/125314857-89583100-e371-11eb-9418-7278eb213a75.png)
#### 접수된 주문에 대한 주문취소 수행
![오더취소 증적4](https://user-images.githubusercontent.com/45377807/125314867-8b21f480-e371-11eb-8c27-0980fc7818db.png)




***

- Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 데이터소스 유형 (RDB or NoSQL) 에 대한 
별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용하였다


		OrderRepository.java
		
		package skhappydelivery;
			
		import org.springframework.data.repository.PagingAndSortingRepository;
		import org.springframework.data.rest.core.annotation.RepositoryRestResource;
			
		@RepositoryRestResource(collectionResourceRel="orders", path="orders")
		public interface OrderRepository extends PagingAndSortingRepository<Order, Long>{
			
			
		}
***

### Req/Res 방식의 서비스 중심 아키텍쳐 구현
#### FeignClient 
- 결제 서비스를 호출하기 위하여 Stub과 (FeignClient) 를 이용하여 Service 대행 인터페이스 (Proxy) 를 구현



		package skhappydelivery.external;
		
		import org.springframework.cloud.openfeign.FeignClient;
		import org.springframework.web.bind.annotation.RequestBody;
		import org.springframework.web.bind.annotation.RequestMapping;
		import org.springframework.web.bind.annotation.RequestMethod;
		
		@FeignClient(name="Pay", url="http://localhost:8085")
		public interface PayService {
		 
		    @RequestMapping(method= RequestMethod.GET, path="/pays")
		    public void pay(@RequestBody Pay pay);
		
		
		    @RequestMapping(method= RequestMethod.POST, path="/payed")
		    public void payed(@RequestBody Payed Payed);
		
		}


- 주문을 받은 직후(@PostPersist) 결제를 요청하도록 처리

		package skhappydelivery;

		import javax.persistence.Entity;
		import javax.persistence.GeneratedValue;
		import javax.persistence.GenerationType;
		import javax.persistence.Id;
		import javax.persistence.PostPersist;
		import javax.persistence.PostUpdate;
		import javax.persistence.Table;
		
		import org.springframework.beans.BeanUtils;
		
		@Entity
		@Table(name="Order_table")
		public class Order {
		
	    @Id
	    @GeneratedValue(strategy=GenerationType.AUTO)
	    private Long orderId;
	    private Long customerId;
	    private String customerName;
	    private String customerAddress;
	    private Integer phoneNumber;
	    private Long menuId;
	    private Integer menuCount;
	    private Integer menuPrice;
	    private Long storeId;
	    private String orderStatus;  
		
	    
	    @PostPersist
	    public void onPostPersist(){

        skhappydelivery.external.Payed Payed = new skhappydelivery.external.Payed();
        // mappings goes here

        Payed.setCustomerId(this.customerId);
        Payed.setOrderId(this.orderId);
        Payed.setStoreId(this.storeId);
        Payed.setTotalPrice(this.menuCount * this.menuPrice);

        OrderApplication.applicationContext.getBean(skhappydelivery.external.PayService.class)
            .payed(Payed);
	    }

#### Istio 구현
<img width="1610" alt="Istio 구현" src="https://user-images.githubusercontent.com/45377807/125317383-ee148b00-e373-11eb-981d-84a7ec4ce2e6.png">


### 이벤트 드리븐 아키텍쳐의 구현 
#### kafka 활용한 Pub/Sub 구조

	package skhappydelivery;
	
	import java.util.Optional;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.cloud.stream.annotation.StreamListener;
	import org.springframework.messaging.handler.annotation.Payload;
	import org.springframework.stereotype.Service;
	
	import skhappydelivery.config.kafka.KafkaProcessor;
	
	@Service
	public class PolicyHandler{
	   @Autowired 
	   private PayRepository payRepository;
		
	   @Autowired
	   private PayService payService;
	
	   @StreamListener(KafkaProcessor.INPUT)
	   public void wheneverOrderCanceled_PayCancel(@Payload OrderCanceled orderCanceled){
	
	   if(!orderCanceled.validate()) return;
	        
	   System.out.println("\n\n##### listener PayCancel : " + orderCanceled.toString() + "\n\n");
	
	   try {
		Optional<Pay> tempObj =  payRepository.findById(orderCanceled.getOrderId());
	
		Pay payObj = new Pay();
	
		if(tempObj.isPresent()){
			payObj = tempObj.get();		
		}else{
			System.out.println("NO PAY data" );
		}
	
			payObj.setPayStatus("ORDERCANCELLED");
	
			payRepository.save(payObj);
	
			System.out.println(" PAYLIST data all :  " + payRepository.findAll().toString());
	
			System.out.println("ORDERCANCELLED SUCCESS");
				
		} catch (Exception e) {

	           System.out.println("\n\n##### listener PayCancel ERROR \n\n");
			
		}
	
	
	
	
	    }//wheneverOrderCanceled_PayCancel
	
	}

![카프카 실행 증적](https://user-images.githubusercontent.com/45377807/125313408-2ca84680-e370-11eb-8828-40ea04e3240c.png)


#### Correlation Key
OrderService.java

	public Order getOrderService(StoreOrderAccepted storeOrderAcceptedObj) throws Exception {

		System.out.println("□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□ getOrderService start "+System.currentTimeMillis()+"□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□□");


		try {
			Optional<Order> tempObj =  orderRepository.findById(storeOrderAcceptedObj.getOrderId());

			Order orderObj = new Order();

			if(tempObj.isPresent()){
				orderObj = tempObj.get();

				orderObj.setOrderStatus(storeOrderAcceptedObj.getOrderStatus());

				orderRepository.save(orderObj);
	
				return orderObj;		
			}else{
				return null ;
			}

		} catch (Exception e) {
			System.out.println("save Order Error" +e.getStackTrace());

			return null;
		}
	}

#### Scaling-out(아래 HPA 참조)


#### 취소에 따른 보상 트랜젝션


#### CQRS


#### Message Consumer


엔티티 패턴과 레포지토리 패턴을 적용하여 JPA를 통한 다양한 데이터소스 유향에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST의 Repository를 적용했다.


### 적용 후 REST API의 테스트
![REST API테스트](https://user-images.githubusercontent.com/45377807/125294258-1645bf00-e35f-11eb-896e-31fb17193885.png)


### Polyglot Programming/Persistence
#### 
#### 
#### 



## 3. 운영

### SLA 준수
#### Pod생성 시 Liveness 와 Readiness Probe를 적용했는가?
#### 셀프힐링: Liveness Probe를 통해 일정 서비스 헬스 상태 저하에 따른 Pod 재생되는지 증명
<img width="1789" alt="Liveness Probe 수행" src="https://user-images.githubusercontent.com/45377807/125291419-59eaf980-e35c-11eb-90f4-edd1130c04c7.png">

#### 서킷브레이커 설정: 서킷브레이커 적용(상단 Istio 구현 참조)


#### 오토스케일러(HPA)
<img width="962" alt="HPA(Autoscaling)_발췌" src="https://user-images.githubusercontent.com/45377807/125291395-5192be80-e35c-11eb-9a6a-a44c133427c8.png">


#### 모니터링, 앨러팅
#### Stateless 한 구현?



### CI/CD 설정
#### AWS Code Build 적용됐는가?

##### buildspec-kubectl.yaml 파일

![빌드스펙yaml파일](https://user-images.githubusercontent.com/45377807/125326441-02a95100-e37d-11eb-8db8-1130577a0cff.png)

##### 빌드 성공
![코드빌드1](https://user-images.githubusercontent.com/45377807/125326080-9e868d00-e37c-11eb-9cdb-093edb64efaf.png)
![코드빌드2](https://user-images.githubusercontent.com/45377807/125326094-a0e8e700-e37c-11eb-8263-0babce52cb25.png)


#### Contract Test
#### (Advanced) Canary Deploy, Shadow Deply, A/B Test (각 2점)
                
### 운영 유연성
#### Config Map / Secret



