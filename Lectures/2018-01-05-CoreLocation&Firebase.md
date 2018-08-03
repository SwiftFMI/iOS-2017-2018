# Frameworks в iOS

Стандартните приложения разработени с iOS позволяват интегриранен на основни (iOS) и външни (open-source, но не само) библиотеки вув вашия iOS App. Днес ще разгледаме няколко основни такива библиотеки, които позволяват да решим специфични задачи, като достъп до местоположението на потребител, достъп до снимките му и комуникация с облачни услуги.

Добър начален пример за представянето на общия процес с достъп до ресурсите на iOS устройстово е достъпа до местоположението на потребителя/устройството.

## CoreLocation

За целта ще се запознаем с онсновната идея, как да поискаме разрешение от потребителя за позлване на хардуера и по-конкретно местположението на му на земята. Това са кооординатите географска ширина и географска дължина (latitude and longitude).

Когато приложение иска да изпозлва определени функции на iOS устройство, тогава потребителя на приложението трябва да се съгласи това да стане. Операционната система отправя запитване и потребителя трябва да отговори позитивно. Ако той откаже, приложението няма да има възможност (по нормален начин) да достъпи желаните ресурси.

### Library

### Privacy Strings 

https://developer.apple.com/documentation/corelocation/cllocationmanager/1620562-requestwheninuseauthorization

### Permission cycle

Кокретните стъпки които трябва да реализираме са следните. Трябва да проверим дали имаме позволение да изпозлваме "ресурса" (локацията). При първоначално инсталиране на приложението, стандартното състояние е неопределено. Когато нямаме достъп или състоянието е неопределено, тогава можем да попитаме потребителя. Тук, напомняме че това питане е добре да се отложи максимално далече във времето, за да можем да направим правилно впречатление на потребителя. Тогава той ще е по-склонен да се съгласи и да даде своето позволение.
Запитването е асинхронен процес, който кара OS-а да визуализира pop-up прозорец, чрез който потребителя може да даде своето съгласие. Ако потребителя е отказал, при всяко следващот запитване ще получаваме автоматичен отказ. Добре е, да отчетем липсата на съгласие и да го подканим да си промени решението, като за целта той ръчно трябва да промени достъпа до "ресурса" през настройки за съответното приложение. Приложенито може да отвори този екран, за да скъси процеса. (Как се отваря този екран? ТБД).


## Firebase (Core) + Database

### Регистграция на приложение

https://console.firebase.google.com/

### CocoaPod 

### Integraciq

### Прочитане на данние

## Photos

Ще реализираме по-модерна версия на UIImagePickerController, която поддържа хоризонтален режим. За целта ще започнем с Master-Detail шаблона. Започваме нов проект, като избираме ```Master-Detail App```. От генерирания проект трябва да направим следните модификации:

1. Да изтрием UITableViewController, който показва детайлите.
2. Да добавим UICollectionViewController
3. Да си направим собствена клетка
	
	картинка на готовата клетка 
	
Добре е да изпозлваме констрейнти за да изглежда добре на различни телефони/екрани. Използваме и различен шрифт за да акцентираме над информацията в клеткта.

За да подредим текстовите полета ги слагаме в ViewStack. Полсе го центрираме хоризонтално в границите на клетката.

Трябва да зададем и статична височина на клетката - 100. Това става от следния екран (снимка).

Дефинираната клетка, ще е базовата, която ще изпозлваме да показваме албумите.

Следващата стъпка е да дефинираме клетката в UICollectionViewController. Тази клетка трябва да съдържа само едно UIImageView за да изобрази снимка. 

Създаваме си собствен клас, който е наследник на клетка.

	КОД

Свързване този клас с клетката в мастър изгледа. (Клетката с двата лейбъла от по-горе.) Свързваме всички елементи на view-то със съответния клас в InterfaceBuilder-a.

Почистваме код-а на view-to (кой клас точно?), който получихме в началото. Крайния резултат може да намерите тук (линк към github файл).


Създаваме си екстеншън на `DetailViewController`, в който имплементираме функциите свързани с UICollectionViewController протокола.

Променяме и основната имплементация на класа.

	добавяме itemSize
	
	initItemSize()
	
	viewDidLayoutSubviews()
	

Създаваме `PhotoLibrary.swift` файл. В него ще дефинираме основните структурни единици - `Photo` и `Album`. Структурата `Photo` ще съдържа информацията за една снимка, а структурата `Album` ще съдържа списък с снимките (`[Photos]`).


## Firebase Storage (Images)

Преди всичко трябва да подготвим приложението, като добавим необходимите библиотеки към проекта. Това става, като включим следния модул към нашия ```Podfile```.

		pod 'Firebase/Storage'

После трябва да инсталираме липсващите пакети през CocoaPods. Това става със следната команада.

		pod install
	
Добре е преди това да сте затворили проекта. След инсталирането пак трябва да отворим workspace файла.

За да изпозлваме Storage функционалността, трябва да конфигурираме правилните права за достъп. [Тук](https://firebase.google.com/docs/storage/security/start?authuser=0) може да прочетем повече за тях.


За предпочитане е да използвате правила, които позволяват качването на файлове само, ако потребителя се е логнал в приложението с потребителско име или парола. За целта на примера, ще позволим на анонимните акунти и те да могат да качват файлове със следните правила:


	// Grants a user access to a node matching their user ID
	service firebase.storage {
	  match /b/{bucket}/o {
	    // Files look like: "user/<UID>/path/to/file.txt"
	    match /user/{userId}/{allPaths=**} {
	      allow read, write: if request.auth.uid == userId;
	    }
	  }
	}

	
Ето и парче код което трябва да модифицираме да работи за логнати потребители в системата. Повече за различните видове на регистрация и активация, може да прочете [тук](https://firebase.google.com/docs/auth/ios/anonymous-auth?authuser=0). Не забравяйте да активирате съответните на следната [страница](https://console.firebase.google.com/u/0/project/fire-base-demo2018/authentication/providers). 

	Auth.auth().signInAnonymously() { [weak self] (user, error) in
            let isAnonymous = user!.isAnonymous  // true
            let uid = user!.uid
            
            print("UID: \(uid)")
            
            let storageRef = Storage.storage().reference()
            
            let imageRef = storageRef.child("user/\(uid)/image.jpg")
        
            // Create file metadata including the content type
            let metadata = StorageMetadata()
            metadata.contentType = "image/png"
            
            if let image = self?.image {
                let data = UIImagePNGRepresentation(image)
                // Upload data and metadata
                let uploadTask = imageRef.putData(data!, metadata: metadata)

                
                uploadTask.observe(.progress) { snapshot in
                    // Upload reported progress
                    let complete = 100.0 * Double(snapshot.progress!.completedUnitCount)
                        / Double(snapshot.progress!.totalUnitCount)
                    
                    print("Progress: \(complete)")
                }

                
                uploadTask.observe(.success) { snapshot in
                    // Upload completed successfully
                    print("File was uploaded.")
                    let url = snapshot.reference.downloadURL { url, error in
                        if let error = error {
                            print("Error: \(error)")
                        } else {
                            print("Public download URL: \(String(describing: url))")
                        }
                    }
                }


                
                uploadTask.observe(.failure) { snapshot in
                    if let error = snapshot.error as NSError? {
                        switch (StorageErrorCode(rawValue: error.code)!) {
                        case .objectNotFound:
                            // File doesn't exist
                            print("object not found")
                            break
                        case .unauthorized:
                            // User doesn't have permission to access file
                            print("user has no permissions")
                            break
                        case .cancelled:
                            // User canceled the upload
                            print("upload was cancelled")
                            break
                            
                            /* ... */
                            
                        case .unknown:
                            // Unknown error occurred, inspect the server response
                            break
                        default:
                            // A separate error occurred. This is a good place to retry the upload.
                            break
                        }
                    }
                }
            }
        }
        
Няколко са основните стъпки. Потребителя се логва в системата като анонимен. Това ни позволява да му създадем профил и да качване снимки в неговия анонимен профил. В бъдеще можем да свържем този анонимен профил с реален.

Създава се файл, който има специален адрес пасващ на правилата от по-горе. Картинката се кодира като PNG за да може да се изпрати. Можем да добавяме произволни записи към метаданните, които да изпозлваме и в правилата и в описанието на ресурса (картинката).

Обекта ```uploadTask``` ни позволява да следми прогреса на качване на ресурса в облака. Можем да паузираме процеса, да го откажем или да следим за успешното приключване. Различни грешки могат да се появят и е добре да ги обработим правилно. Когато ресурът е качен, можем да генерираме публичен URL, който да бъде иползван извън нашето приложение. Този URL може да бъде премахнат когато решим и през портала на Firebase. Това дава голяма свобода да свързваме тези ресурси с данните от базата.

Записките представят само начален поглед над Firebase облачните услуги предоставени от Google. За да постигнете повече с тях е добре да познвате пълния арсенал и какви стандартни задачи ви позволява. Ето няколко интересни точки:
* нотификации
* отдалечено конфигуриране
* крашлитикс (дкладване на крашове)
* аналитикс (статистика)
* реклама, чрез AdWords 