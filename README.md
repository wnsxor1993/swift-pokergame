# swift-pokergame
## 게임보드 만들기
### 프로그래밍 요구사항
- 앱 기본 설정을 지정해서 StatusBar 스타일을 LightContent로 보이도록 한다.

<img src = "https://user-images.githubusercontent.com/44107696/154883554-2abc18cc-5da6-4233-8ab5-75bea973ef81.png" width="900" height="1190">
    * 다양한 속성 관련 Key 값들

<img src = "https://user-images.githubusercontent.com/44107696/154883533-1254ab12-79f7-43e7-b475-7aedb7413e34.png" width="800" height="700">
    * View controller-based status bar appearance 값을 false로 설정 (상태 표시줄 모양이 현재의 VC가 택한 스타일을 기반으로 할 것인지에 대한 Key이므로 Info 수정 시에는 필수로 false 필요)
    * Info plist가 아닌 preferredStatusBarStyle 프로퍼티를 오버라이드해서 VC 내에서 설정하는 방법 또한 존재
    * Status bar style Key의 값을 Light Content로 수정. 현재는 흰 배경이라 status bar가 묻혀서 보이지 않는 상태

- ViewController 클래스에서 self.view 배경을 다음 이미지 패턴으로 지정한다. 이미지 파일은 Assets에 추가한다.
- 다음 카드 뒷면 이미지를 다운로드해서 프로젝트 Assets.xcassets에 추가한다.

<img src = "https://user-images.githubusercontent.com/44107696/154883563-efe647f3-b152-45b6-888f-4792793dc2b9.png" width="940" height="810">
    * backgroundColor 설정 중 UIColor(patternImage:)를 활용하여 Asset의 이미지를 패턴화시켜 UIColor로 변환하여 적용
    * 배경 변경 후에는 Light Content로 수정된 StausBar도 색이 묻히지 않고 잘 표시됨

- ViewController 클래스에서 코드로 7개 UIImageView를 생성하고, 추가해서 카드 뒷면을 보여준다
    + 주의사항 우선 스택뷰StackView를 사용하지 말고 직접 UIImageView를 7개 생성해야 한다
- 화면 크기를 구하고 균등하게 7등분해서 이미지를 표시해야 한다
- 카드 가로와 세로 비율은 1:1.27로 지정한다

<img src = "https://user-images.githubusercontent.com/44107696/154883565-d531b979-8279-48a0-8fa4-fe496c7a52ec.png" width="1000" height="900">
    * class 프로퍼티로 카드 뒷면 Image, 값 변경이 가능한 xPosition 선언
    * ImageView 생성 함수 구현. 13pro의 가로값 390을 7개로 분할할 때, 카드 너비 54 / 카드 간 간격 2로 설정하면 정확히 나눠지므로 이를 토대로 카드의 너비와 높이 할당
    * frame을 통한 위치 설정 중 y 값은 50으로 고정
    * 7개의 카드를 만들기 위해 viewDidLoad에서 반복문을 통해 함수 호출

### 추가 구현사항
- 앱 기본 설정(Info.plist)을 변경하는 방식에 대해 학습하고 앱 표시 이름을 변경한다

<img src = "https://user-images.githubusercontent.com/44107696/154883567-2a4e1066-ae91-4d9f-a354-ba8afefaed30.png" width="800" height="890">
    * Info plist에서 BundleName 추가. 시뮬레이터 홈 화면에서 변경된 Yu-Gi-Oh!로 앱 이름 표시되는 것을 확인
    
- Stack View 추가
    + 추가 후 코드 변경되기 전의 코드
    ```swift
    class ViewController: UIViewController {
        private let cardImage = UIImage(named: "card-back")
        private var cardXPosition: Double = 0
        
        override func viewDidLoad() {
            super.viewDidLoad()
            
            if let pattern = UIImage(named: "bg_pattern"){
                self.view.backgroundColor = UIColor(patternImage: pattern)
            }
            
            for _ in 0...6{
                makeImageView()
            }
        }

        func makeImageView(){
            let cardWidth: Double = 54
            let cardHeight: Double = cardWidth * 1.27
            
            let imageView = UIImageView(frame: CGRect(x: self.cardXPosition, y: 50, width: cardWidth, height: cardHeight))
            imageView.image = self.cardImage
            
            // 나열을 위해 x 값 변경 (카드 가로 54 / 카드 간격 2 ; 13pro 가로 390)
            self.cardXPosition += 56
            
            self.horizonStack.addSubview(imageView)
        }
    }
    ```
    
    + Stack View에 맞춰서 코드 변경
        * StackView는 Auto layout을 적용해 내부에 배치된 view들을 배치하므로 subview 개개로 크기 등의 요소를 조절하는 것이 의미 없어지므로 makeImageView 함수는 imageView를 image 매개변수로 생성하고 addArrangedSubview를 통해 StackView의 stack에 추가하는 로직으로 수정
        * StackView의 높이는 7개의 subview 각각의 너비에 1.27배로 계산 (기존 카드의 너비 대비 높이가 나올 수 있도록). subview 간의 간격은 3, 동일 크기로 stackview를 채우도록 fill equally로 설정.
        
    ```swift
    class ViewController: UIViewController {
        private let cardImage = UIImage(named: "card-back")
        
        @IBOutlet weak var horizonStack: UIStackView!
        
        override func viewDidLoad() {
            super.viewDidLoad()
            
            horizonStack.frame.size.height = horizonStack.frame.width / 7 * 1.27
            horizonStack.distribution = .fillEqually
            horizonStack.spacing = 3
            
            if let pattern = UIImage(named: "bg_pattern"){
                self.view.backgroundColor = UIColor(patternImage: pattern)
            }
            
            for _ in 0...6{
                makeImageView()
            }
        }

        func makeImageView(){
            let imageView = UIImageView(image: self.cardImage)
            
            self.horizonStack.addArrangedSubview(imageView)
        }
    }
    ```


## 카드 클래스 구현하기
### 프로그래밍 요구사항
- 객체지향 프로그래밍 방식에 충실하게 카드 클래스(class)를 설계한다.
    + 속성으로 모양 4개 중에 하나, 숫자 1-13개 중에 하나를 가질 수 있다.
    + 모양이나 숫자도 적당한 데이터 구조로 표현한다. 클래스 혹은 Nested enum 타입으로 표현해도 된다.
    + 어떤 이유로 데이터 구조를 선택했는지 주석(comment)으로 구체적인 설명을 남긴다.
    + 카드 정보를 출력하기 위한 문자열을 반환하는 함수를 구현한다.
    + 문자열에서 1은 A로, 11은 J로, 12는 Q로, 13은 K로 출력한다.
    
    ```swift
    struct CardNumber{
        private let number: Int
        var changedNumber: String{
            switch number{
            case 1:
                return "A"
            case 11:
                return "J"
            case 12:
                return "Q"
            case 13:
                return "K"
            default:
                return number.description
            }
        }
        
        init(num: Int){
            number = num
        }
    }

    enum Shape: String{
        case heart = "♥️", case dia = "♦️", case spade = "♠️", case clover = "♣️"
    }
    ```
    * 숫자 값은 변환 값 외에 실제 숫자를 사용하는 경우도 있으므로 모두 보관할 수 있는 Struct로 고려 (enum은 상기의 목적을 이루기 위해 모든 숫자 나열 필요, 클래스는 참조나 고유 값 등이 필요하지 않으므로)
    * 문양은 4가지 밖에 안 되는 간단한 상황이므로 enum으로 고려
    
    ```swift
    class Card{
        private let number: CardNumber
        private let shape: Shape
        
        func cardInformation() -> String{
            let cardValue = shape.rawValue + number.changedNumber
            return cardValue
        }
        
        init(number: CardNumber, shape: Shape){
            self.number = number
            self.shape = shape
        }
    }
    ```
    * 문양과 숫자 정보를 합쳐서 리턴해주는 cardInformation 함수

- ViewController에서 특정한 카드 객체 인스턴스를 만들어서 콘솔에 출력한다
- 데이터를 처리하는 코드와 출력하는 코드를 분리한다
    + 랜덤으로 문양과 숫자를 선택해서 카드 인스턴스를 만들어주는 makeCardInfo 함수 구현
    + 전달받은 카드 인스턴스의 정보를 출력해주는 showCardInfo 함수 구현
    
    ```swift
    class ViewController: UIViewController {
        private let cardImage = UIImage(named: "card-back")
        
        @IBOutlet weak var horizonStack: UIStackView!
        
        override func viewDidLoad() {
            super.viewDidLoad()
            
            horizonStack.frame.size.height = horizonStack.frame.width / 7 * 1.27
            horizonStack.distribution = .fillEqually
            horizonStack.spacing = 3
            
            if let pattern = UIImage(named: "bg_pattern"){
                self.view.backgroundColor = UIColor(patternImage: pattern)
            }
            
            for _ in 0...6{
                makeImageView()
                let cardInfo = makeCardInfo()
                showCardInfo(card: cardInfo)
            }
        }

        func makeImageView(){
            let imageView = UIImageView(image: self.cardImage)

            self.horizonStack.addArrangedSubview(imageView)
        }
        
        func makeCardInfo() -> Card{
            let shapes: [Shape] = [.clover, .dia, .heart, .spade]
            let shapeRandomNum = Int.random(in: 0...3)
            let cardRandomNum = Int.random(in: 1...13)
            
            let cardInfo = Card(number: CardNumber(num: cardRandomNum), shape: shapes[shapeRandomNum])
            
            return cardInfo
        }
        
        func showCardInfo(card: Card){
            print(card.cardInformation())
        }
    }
    ``` 
    
<img src = "https://user-images.githubusercontent.com/44107696/155056906-3b59e657-5364-4444-8a31-51cdc8f0c61e.png" width="800" height="800">

### 수정사항 - (1)
```swift
// 하나의 카드가 가지게 될 Card 인스턴스는 항상 고유한 값이 되어야 하므로 class로 설계
class Card: CustomStringConvertible{
    private let number: Card.CardNumber
    private let shape: Card.Shape
    
    var description: String{
        let cardValue = shape.description + number.description
        return cardValue
    }
    
    // 랜덤 문양, 숫자의 카드 인스턴스 초기화
    init(){
        guard let shapeRandomNum = Shape.allCases.randomElement() else{
            print("적절한 Case 값을 찾지 못하였습니다.")
            fatalError()
        }
        
        let cardRandomNum = Int.random(in: 1...13)
        
        self.number = CardNumber(num: cardRandomNum)
        self.shape = shapeRandomNum
    }
    
    // 변경해줘야 하는 값의 종류가 다소 많은 편이며, 카드의 값을 오로지 숫자로만 사용하는 놀이 등도 존재하므로 숫자와 문자열 모두 보관할 수 있는 형태를 생각함. 고유한 값으로 존재하거나 꼭 참조될 필요가 없으므로 Struct로 표현
    struct CardNumber: CustomStringConvertible{
        private let number: Int
        var description: String{
            switch number{
            case 1:
                return "A"
            case 11:
                return "J"
            case 12:
                return "Q"
            case 13:
                return "K"
            default:
                return number.description
            }
        }
        
        init(num: Int){
            guard num <= 13 || num != 0 else{
                print("적절하지 못한 값이 발견되었습니다 : \(num)")
                fatalError()
            }
            
            number = num
        }
    }

    // 4가지 문양밖에 없는 간단한 변환 처리이므로 enum으로 간소하게 표현
    enum Shape: CaseIterable, CustomStringConvertible{
        case heart, dia, spade, clover
        var description: String{
            switch self {
            case .heart:
                return "♥️"
            case .dia:
                return "♦️"
            case .spade:
                return "♠️"
            case .clover:
                return "♣️"
            }
        }
    }
}
``` 

- Card 인스턴스의 주요 프로퍼티의 타입인 CardNumber와 Shape은 Card 클래스와의 관계성이 높으며, 차후 재사용 가능성이 낮으므로 nested 형태로 수정
- CustomStringConvertible을 채택하고 description의 오버라이드를 통해 속성을 문자열 값으로 전달하도록 모든 커스텀 속성들 수정
- CardNumber는 인자로 Int를 받으니 Int.random 사용. Shape은 가지고 있는 속성들을 random으로 선택하는 로직을 구현하기 위해 caseIterable 프로토콜 채택 (케이스들을 순회할 수 있도록 만들어주며, allCases라는 모든 케이스를 배열로 담은 프로퍼티를 호출할 수 있음)
- guard 구문을 활용하여 예외사항에 대한 alert를 콘솔창에 띄워주며 더이상 작업이 진행되지 못하도록 fatalError 호출
- VC 내에서는 Card 인스턴스를 인자 없이 생성 가능하게 되었고, 출력 함수또한 description을 호출하는 형태로 변경


### 수정사항 - (2)
```swift
// 하나의 카드가 가지게 될 Card 인스턴스는 항상 고유한 값이 되어야 하므로 class로 설계
class Card: CustomStringConvertible{
    private let number: Card.CardNumber
    private let shape: Card.Shape
    
    var description: String{
        let cardValue = shape.description + number.description
        return cardValue
    }
    
    // 랜덤 문양, 숫자의 카드 인스턴스 초기화
    init(number: Card.CardNumber, shape: Card.Shape){
        self.number = number
        self.shape = shape
    }
    
    
    // 한정된 범위에 맞춰 enum 타입으로 수정
    enum CardNumber: CaseIterable ,CustomStringConvertible{
        case one, two, three, four, five, six, seven, eight, nine, ten, eleven, twelve, thirteen
        var description: String{
            switch self {
            case .one: return "A"
            case .two: return "2"
            case .three: return "3"
            case .four: return "4"
            case .five: return "5"
            case .six: return "6"
            case .seven: return "7"
            case .eight: return "8"
            case .nine: return "9"
            case .ten: return "10"
            case .eleven: return "J"
            case .twelve: return "Q"
            case .thirteen: return "K"
            }
        }
    }

    // 4가지 문양밖에 없는 간단한 변환 처리이므로 enum으로 간소하게 표현
    enum Shape: CaseIterable, CustomStringConvertible{
        case heart, dia, spade, clover
        var description: String{
            switch self {
            case .heart:
                return "♥️"
            case .dia:
                return "♦️"
            case .spade:
                return "♠️"
            case .clover:
                return "♣️"
            }
        }
    }
}
``` 

```swift
class ViewController: UIViewController {
    private let cardImage = UIImage(named: "card-back")
    
    @IBOutlet weak var horizonStack: UIStackView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        horizonStack.frame.size.height = horizonStack.frame.width / 7 * 1.27
        horizonStack.distribution = .fillEqually
        horizonStack.spacing = 3
        
        if let pattern = UIImage(named: "bg_pattern"){
            self.view.backgroundColor = UIColor(patternImage: pattern)
        }
        
        for _ in 0...6{
            makeImageView()
            
            if let cardInfo = makeRandomCardInfo(){
                showCardInfo(card: cardInfo)
            } else{
                let alert = UIAlertController(title: "잘못된 호출", message: "모양이나 숫자 입력이 잘못되었습니다. 다시 확인해주세요.", preferredStyle: UIAlertController.Style.alert)
                self.present(alert, animated: true)
            }
        }
    }

    func makeImageView(){
        let imageView = UIImageView(image: self.cardImage)

        self.horizonStack.addArrangedSubview(imageView)
    }
    
    func makeRandomCardInfo() -> Card?{
        guard let cardRandomShape = Card.Shape.allCases.randomElement(), let cardRandomNum = Card.CardNumber.allCases.randomElement() else{
            return nil
        }
        
        let cardInfo = Card(number: cardRandomNum, shape: cardRandomShape)
        
        return cardInfo
    }
    
    func showCardInfo(card: Card){
        // description 생략 시에도 description을 가져와서 출력
        print(card)
    }
}
``` 

- CardNumber 또한 한정된 범위를 지니고 있으므로 enum 타입으로 수정
- 범용성이 떨어졌던 이전 수정의 이니셜라이저는 상위의 값을 전달받아 할당해주는 형태로 수정. (차후에는 Int, String 등을 받은 걸 변환시켜 할당하는 요소도 추가 고려 중)
- makeCardInfo 함수는 makeRandomInfo 함수로 이름을 변경, 이에 맞춰 적절한 범위 내의 랜덤 값을 변환하여 Card에 값을 전달하도록 수정. 잘못된 값이 들어갈 경우 nil로 전달하도록 리턴 타입은 Card?
- Card 인스턴스 생성 시, nil이 들어갈 경우 alert를 통해서 재입력 부탁 메세지 출력


## 카드덱 구현하고 테스트하기
### 기능 요구사항
- 앞서 만든 모든 종류의 카드 객체 인스턴스를 포함하는 카드덱 구조체를 구현한다.
- 객체지향 설계 방식에 맞도록 내부 속성을 모두 감추고 다음 인터페이스만 보이도록 구현한다.
    + count 갖고 있는 카드 개수를 반환한다.
    + shuffle 기능은 전체 카드를 랜덤하게 섞는다.
    + removeOne 기능은 카드 인스턴스 중에 하나를 반환하고 목록에서 삭제한다.
    + reset 처음처럼 모든 카드를 다시 채워넣는다.

    ```swift
    class ViewController: UIViewController {
        private let cardImage = UIImage(named: "card-back")
        
        @IBOutlet weak var horizonStack: UIStackView!
        
        override func viewDidLoad() {
            super.viewDidLoad()
            
            horizonStack.frame.size.height = horizonStack.frame.width / 7 * 1.27
            horizonStack.distribution = .fillEqually
            horizonStack.spacing = 3
            
            if let pattern = UIImage(named: "bg_pattern"){
                self.view.backgroundColor = UIColor(patternImage: pattern)
            }
            
            for _ in 0...6{
                makeImageView()
                
                if let cardInfo = makeRandomCardInfo(){
                    showCardInfo(card: cardInfo)
                } else{
                    let alert = UIAlertController(title: "잘못된 호출", message: "모양이나 숫자 입력이 잘못되었습니다. 다시 확인해주세요.", preferredStyle: UIAlertController.Style.alert)
                    self.present(alert, animated: true)
                }
            }
        }

        func makeImageView(){
            let imageView = UIImageView(image: self.cardImage)

            self.horizonStack.addArrangedSubview(imageView)
        }
        
        func makeRandomCardInfo() -> Card?{
            guard let cardRandomShape = Card.Shape.allCases.randomElement(), let cardRandomNum = Card.CardNumber.allCases.randomElement() else{
                return nil
            }
            
            let cardInfo = Card(number: cardRandomNum, shape: cardRandomShape)
            
            return cardInfo
        }
        
        func showCardInfo(card: Card){
            // description 생략 시에도 description을 가져와서 출력
            print(card)
        }
    }
    ``` 
    * CardDeck에는 52장의 카드를 담을 deck 프로퍼티와 드로우된 카드들을 기록하는 removedCards로 구현 (덱에서 나간 카드들은 현실에서도 어떤 카드인지 공개되지 않으므로 private 설정 후, 어떤 메서드로도 내부를 볼 수 있는 로직 구현하지 않음)
    * count 구현, 보통 개수는 출력과 관련되는 경우가 많으므로 return 값 설정
    * shuffle 구현, Fisher-Yates shuffle 알고리즘에 맞춰 랜덤 값으로 나온 인덱스의 value를 신규 덱에 넣고 기존 덱에서는 삭제. 이를 기존 덱이 빌 때까지 반복한 뒤, 반복이 끝나면 신규 덱을 기존 덱에 할당
    * reset 구현, 빠진 카드들을 다시 주워담아서 합치는 형태로 구현했으며 빠진 카드가 없으면 괜히 로직이 실행되지 않도록 guard 구문 작성
    * removeOne 구현, 보통 카드를 드로우할 때에는 맨 뒷장이 빠지므로 removeLast로 구현. 남은 카드가 없을 수도 있으므로 리턴은 옵셔널 Card
    * 카드 구성은 기본적으로 52장 세팅이므로 4개의 문양 별로 13장의 카드 생성 로직 구성

- 상위 모듈에서 카드덱 기능을 확인할 수 있도록 테스트 코드를 추가한다
- XCTest 같은 단위 테스트가 아니라, 카드덱 함수를 호출해서 원하는 데로 동작하는지 확인할 수 있는 코드를 작성한다
    + 테스트 시나리오와 비슷한 형태로 테스트 구문 구현 ( guard 조건 시, 아래 구문 미실행 부분들도 추가로 검토 진행함 )
    
    ```swift
    func testCardDeck(){
        // 카드 초기화
        var deck = CardDeck()
        // 초기화 카드 수 52장 확인
        let firstDecks = deck.count()
        
        // 카드 셔플
        deck.shuffle()
        
        deck.reset()
        
        // 카드 뽑기
        let card1 = deck.removeOne()
        let secondDecks = deck.count()
        
        // 카드 또뽑기
        let card2 = deck.removeOne()
        let thirdDecks = deck.count()
        
        // 카드 리셋
        deck.reset()
        let fourthDecks = deck.count()
    }
    ```

<img src = "https://user-images.githubusercontent.com/44107696/155133976-84732446-1b2b-4fbf-8ec3-169ef4477fbc.png" width="800" height="800">

