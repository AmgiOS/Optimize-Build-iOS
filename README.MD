Quelques conseils simples pour optimiser le temps de compilation d'un projet iOS.

# Prérequis - Afficher le temps de Build
- Tout d'abord, nous devons savoir combien de temps Xcode prend pour construire notre projet. Par défaut, Xcode n'affiche pas le temps de construction, nous devrons donc l'activer.

1. Copiez-collez simplement et exécutez cette commande dans le terminal :
```
defaults write com.apple.dt.Xcode ShowBuildOperationDuration -bool YES
```

2. Maintenant, après avoir construit votre projet (Command + B), la barre de Xcode devrait afficher le temps de construction comme ceci :

![Capture_d_écran_2021-11-25_à_11.03.28](uploads/dda5b9bf11188bb90e82b4f360a980b7/Capture_d_écran_2021-11-25_à_11.03.28.png)

Remarque 1 :  si Xcode ne l'affiche pas pour une raison quelconque, essayez de le redémarrer

Remarque 2:  Afin d'afficher le temps de construction correct, assurez-vous de nettoyer en profondeur votre projet. Les données dérivées, y compris le dossier Build (Command + Maj + K) doivent être nettoyées avant de générer le projet.

# Build System

- Xcode 9 a introduit un nouveau système de build, celui-ci est écrit entièrement en Swift et donc plus optimiser. L'objectif principal de ce nouveau système de construction est de réduire le temps de build global.

1. Pour utiliser le nouveau système de build, vous pouvez l'activer dans le menu "File" et sélectionner "Workspace Settings".

![Xcode-New-Build-System-cgd25](uploads/c5ba3cf6b0319c24de8d3940af8fa474/Xcode-New-Build-System-cgd25.png)

# Architecture Impact

- Les devices et les simulateurs ont une architecture différente, mais généralement, pendant le développement, nous voulons que Xcode exécute la construction sur l'appareil ou sur le simulateur uniquement. Ainsi, en définissant 'Build Active Architecture Only' sur 'Yes', nous demandons au compilateur de générer le binaire uniquement pour une architecture. Cela aura un impact drastique en fonction des changements dans les modules et les dépendances de votre projet.

1. Accédez à « Build Active Architecture Only » dans les paramètres de construction de votre projet et définissez « Debug » sur « Oui » et « Release » sur « Non ».

![Xcode-Active-Architecture-fimat](uploads/e5bf3bf63f3f8a48fa3e3eb7a091525c/Xcode-Active-Architecture-fimat.png)

# Niveau d'optimisation

- Le « Optimization Level » indique au compilateur d'optimiser le build à un certain niveau. Généralement, les versions de débogage sont définies avec « No Optimization » car elles permettent aux développeurs de déboguer via les valeurs contenues par let/var. Ceci est très nécessaire pendant la phase de débogage.

1. Accédez à « Optimization Level » dans les Build Settings et vérifiez les valeurs définies :

![Xcode-Optimization-Level-1180x199-05jfx](uploads/22a906ff5e1b345f78ac2a70ffa154c7/Xcode-Optimization-Level-1180x199-05jfx.png)

2. Si vous ne faites pas beaucoup de débogage, il est préférable de régler ce paramètre sur « Optimiser pour la vitesse ». Cela réduira éventuellement le temps de génération car le compilateur omettra les étapes d'attachement de valeurs au thread du débogueur.

-------------------

# Optimisations du code:

## 1. Identifiez le code qui se compile lentement
 
- Xcode nous permet d'identifier les blocs de code qui causent d'énormes retards dans le temps de compilation. Vous pouvez spécifier une limite de temps pour l'exécution d'un bloc de code et laisser Xcode lancer un avertissement pour le code qui dépasse la limite de temps spécifiée.

**Sous « Other Swift Flags », ajoutez ces lignes :**
```
-warn-long-function-bodies=200
-warn-long-expression-type-checking=200
```
![Xcode-Identify-Lazy-Code-1180x100-lyk4w](uploads/42bd85d85be519ea4f2966364dda3c5d/Xcode-Identify-Lazy-Code-1180x100-lyk4w.png)

2. La valeur « 200 » représente la limite en millisecondes. Ainsi, une fois la construction exécutée, Xcode lancera des avertissements pour toute fonction ou expression qui prend plus de 200 ms pour être exécutée.

## 2. Conseils sur le code

**Supprimez le code vierge inutile:**

Le code qui ne fait rien du tout est également compilé par le compilateur, c'est donc une bonne idée de supprimer ce code.

```
final class FilterCell: UITableViewCell {

    @IBOutlet weak var labelTitle: UILabel!
    
/* Ce code la sera compilé alors que celui-ci n'est pas utilisé

    override func awakeFromNib() {
        super.awakeFromNib()
    }

    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
    }
*/
}
```

**Utiliser une constante ("let") autant que possible**

```
final class ViewController: UIViewController {
  private let constantWidth: CGFloat = 100
  private let constantHeight: CGFloat = 50
}
```

**Utilisez l'access control et mettez le maximum de classes et propriétés en "private", "final" et "fileprivate" également pour les extensions**

```
final class ViewController: UIViewController {
  private let constantWidth: CGFloat = 100
  private let constantHeight: CGFloat = 50
  fileprivate var isUpdating: Bool = false
  private(set) var isDataAvailable: Bool = false
  @IBOutlet private var textLabel: UILabel!

  @objc private func gestureRecognized() {
  }
}
```

**Spécifiez clairement le type**

- Ne laissez pas le compilateur déduire.
Si vous ne spécifiez pas de types, le compilateur doit de lui meme déduire les types, ce qui prend du temps. Il est préférable de spécifier le type pour réduire le travail du compilateur.

```
final class ViewController: UIViewController {
  private let constantWidth: CGFloat = 100
  fileprivate var isUpdating: Bool = false
  private(set) var isDataAvailable: Bool = false
  private var userNames: [String] = [“Amit”, “Yogesh”, “Rohit”]
  var numbers: [Int] = [1, 2, 3]
  private var savedPaymentMethods: [SavedPaymentMethod] = []
}
```
- **Évitez les types Objective-C:**

```
public class ViewController: UIViewController {
    var dictionary = Dictionary<String, Any>()
    var nsDictionary = NSMutableDictionary()
    var array = Array<String: Any>()
    var nsArray = NSMutableArray()
    var anyObject: AnyObject?
}
```
**Évitez l'interpolation de chaîne dans la mesure du possible:**

- La concaténation de chaînes se compile plus rapidement que l'interpolation. Utilisez donc la concaténation lorsque toutes les variables qui doivent être concaténées sont de type String.

```
var storageIdentifier: String {
    return "\(storageName)_\(installationId)"
}
```

- **Évitez les fonctions longues**
```
override func viewDidLoad() {
    super.viewDidLoad()
    // .... some other huge code of lines
    // .... some other huge code of lines
    // .... some other huge code of lines
    // .... some other huge code of lines
    // .... some other huge code of lines
    // .... some other huge code of lines
    // .... some other huge code of lines
    title = detailState.navTitle
    navigationController?.navigationBar.titleTextAttributes = [.font: UIFont.systemFont(ofSize: 20)]
    navigationController?.navigationBar.tintColor = .lightGray
    validationController = SomeValidationController(fieldProvider: self, detailState: detailState)
    validationView.delegate = self
    // .... some other huge code of lines
    // .... some other huge code of lines
    // .... some other huge code of lines
    // .... some other huge code of lines
    // .... some other huge code of lines
    // .... some other huge code of lines
}
````
# Optimisation Storyboard:

**Supprimez les contrôleurs de storyboard inutilisés**

- Le storyboard est le principal coupable du ralentissement du compilateur. Dans la plupart des cas où il n'y a qu'un seul storyboard, cela prend environ 40 à 60% du temps de compilation total.

- Une façon de réduire ce temps consiste à supprimer les contrôleurs inutilisés ou peut-être à les déplacer vers un storyboard séparé et à décocher leur appartenance cible afin qu'ils ne soient pas compilés.
Si nous avons des écrans/contrôleurs qui ne sont pas actuellement utilisés, déplacez-les vers inutilisés.

- Un seul storyboard prend énormément de temps à compiler. Si nous le divisons en plusieurs storyboards, ils seront tous compilés en parallèle et feront une grande différence dans le temps de compilation. Il est toujours préférable de diviser les storyboards.


![image](uploads/983c8e69f7c40e99c998aefee74c4bc9/image.png)

**Évitez d'avoir trop d'éléments dans un seul contrôleur:**

- L'ajout de trop de vues dans un seul contrôleur prend énormément de temps à compiler. Vous trouverez ci-dessous un exemple où un seul contrôleur contient trop de vues. Nous devrions éviter cela et essayer de le diviser en plusieurs contrôleurs pour améliorer le temps de compilation.

![image](uploads/c56b3c1a3a5eca910b2f9cfa27941b13/image.png)

# Choix du Gestionnaire de dépendances

Pour finir cette documentation, voici une explication sur Cocoapods et Carthage, sur leur différences et pourquoi il serait plus judicieux de choisir l'un ou l'autre sur un projet. Car chacun à sa particularité sur sa gestion propre des librairies.

## CocoaPods contre Carthage
- En bref, Carthage est un meilleur choix si vous effectuez des constructions propres fréquentes et que vous vous souciez du temps de construction.

### Inconvénient des CocoaPods
- CocoaPods entraîne des temps de compilation plus longs, car le code source des bibliothèques tierces est dans la plupart des cas compilé chaque fois que vous effectuez une nouvelle génération. En général, vous ne devriez pas avoir à le faire souvent, mais en réalité, vous le faites.

### Avantage Carthage
- Il ne se reconstruit jamais sur une version propre, vous ignorez donc tous les temps de construction des dépendances. Vous créez des dépendances externes uniquement lorsque vous modifiez quelque chose dans la liste des dépendances (ajoutez un nouveau framework, mettez à jour un framework vers une version plus récente, etc.). Cela peut prendre un certain temps, mais vous le faites beaucoup moins souvent que lorsque vous créez du code intégré à CocoaPods. 

Voici les sources utilisées pour la création de cette doc:
- https://medium.com/@RobertGummesson/regarding-swift-build-time-optimizations-fc92cdd91e31
- https://medium.com/swift-programming/swift-build-time-optimizations-part-2-37b0a7514cbe
- https://betterprogramming.pub/improve-xcode-compile-and-run-time-8b8f812c17f8
- https://betterprogramming.pub/improve-xcode-compile-and-run-time-8b8f812c17f8
- https://flexiple.com/ios/xcode-build-optimization-a-definitive-guide/