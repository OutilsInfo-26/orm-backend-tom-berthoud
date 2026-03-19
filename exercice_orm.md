# Exercice : pratiquer ORM + SQL brut (FastAPI)

Objectif : étendre le mini projet ORM pour manipuler des requêtes SQL et ORM, tout en gardant la validation via schemas Pydantic.

---

## Questions de compréhension

Répondez aux questions suivantes avant de commencer l'implémentation.

### Base de données

1. Qu'est-ce qu'une clé primaire et à quoi sert-elle ?  C'est un id qui va être unique pour chaque élèment de la table

2. Qu'est-ce qu'une clé étrangère ? Donnez un exemple avec les tables du projet.    C'est l'id d'une autre table qui va permettre de faire des liaisons. Par exemple, book possède author_id.


3. Quelle est la différence entre un `INNER JOIN` et un `LEFT JOIN` ? Quand utilise-t-on l'un plutôt que l'autre ? Inner join n'affiche pas les id qui pointent vers un id inexistant alors que left les affiches mais avec NULL.


4. Qu'est-ce qu'une table de jointure (association) ? Pourquoi en utilise-t-on une dans ce projet ?

 Une table qui va relier des relations N:M.

5. Quelle est la différence entre une relation 1:N et une relation N:M ? une relation 1:N n'as pas besoin d'une table d'associations.



### Modèles SQLAlchemy (`models.py`)

1. Pourquoi crée-t-on une classe `Base` qui hérite de `DeclarativeBase` au lieu d'hériter directement de `DeclarativeBase` dans chaque modèle ? 

Car autrement chaque modele serait sa propre base de déclaration, Si on crée la table on ne pourrait pas créer directement chaque model.
   
2. Comment sont créées les tables en base de données à partir des modèles Python ? 
   
    On crée des modèles à l'aide de la bibliothjèque DeclarativeBase et il va automatiquemetn créer les tables dans la base de donnée

3. Expliquez cette ligne :
   ```python
   id: Mapped[int] = mapped_column(primary_key=True)
   ```
   Cela indique de crée une colone qui est la clé primaire (l'id).
4.  Comment indique-t-on qu'un champ peut être `NULL` (optionnel) en base de données avec SQLAlchemy 2.0 ?  Il faut mettre Mapped[int | None]
   
5.  Expliquez cette ligne :
    ```python
    book_tags: Mapped[list["BookTag"]] = relationship("BookTag", back_populates="book")
    ```
    Que signifie `back_populates` ? Que se passe-t-il si on l'omet ?
    `back_populates` peut se traduire par remplit le lien en retour, il faut mettre le nom du lien présent dans l'autre table. Si on l'enlève il y aura soit une erreur, soit on perd la possiblité de fais des liens directe comme books.tag

6.  Dans le modèle `Book`, `publisher_id` est défini comme `ForeignKey` mais il n'y a pas de `relationship` vers `Publisher`. Quelle conséquence cela a-t-il sur les requêtes ?
    Cela veut dire que pour faire une jointure entre Book et Publisher, il faudra le faire manuellement dans les requêtes, on ne pourra pas faire book.publisher.name par exemple.

### Schemas Pydantic (`schemas.py`)

12. À quoi servent les schemas Pydantic dans ce projet ? Pourquoi ne retourne-t-on pas directement les objets SQLAlchemy ?
    Les schémas servent à valider les données reçu depuis les requêtes et à formater les données retournées par l'API.
13. Dans `BookCreate`, expliquez le rôle des `...` dans `Field(...)` et celui de `min_length`, `max_length`.
    Les `...` indiquent que le champ est obligatoire, `min_length` et `max_length` sont des contraintes de validation qui indiquent la longueur minimale et maximale d'une chaîne de caractères.

14. Qu'est-ce que `model_config = {"from_attributes": True}` et dans quel cas est-ce nécessaire ?
    Cela indique que le schéma peut être créé à partir d'un objet SQLAlchemy, en mappant les attributs de l'objet aux champs du schéma. C'est nécessaire pour pouvoir retourner des objets SQLAlchemy directement dans les routes FastAPI et les faire convertir automatiquement en schémas Pydantic.

15. Quelle est la différence entre `BookCreate` et `BookOut` ? Pourquoi avoir deux schemas séparés ?
    
    `BookCreate` est utilisé pour valider les données reçues lors de la création d'un livre, tandis que `BookOut` est utilisé pour formater les données retournées par l'API. Avoir deux schémas séparés permet de différencier les contraintes de validation et les champs nécessaires pour la création d'un livre de ceux qui sont retournés lors de la lecture d'un livre.

16. Dans `AuthorUpdate`, tous les champs sont optionnels (`str | None`). Pourquoi ? Quelle est la différence avec `AuthorCreate` ?
    
Car si on fait une update, on peut ne pas modifier tout les champs.

### Routes FastAPI (`orm_simple.py`, `orm_join.py`, etc.)

17. Si le router est défini avec `prefix="/orm"`, pourquoi faut-il appeler `/orm/authors` et non `/authors` ?
    Car il y aurait une ambiguïté avec les routes définies dans `sql_simple.py` qui ont aussi des routes `/authors`. Le préfixe permet de différencier les routes liées à l'ORM de celles liées au SQL brut.

18. Quelle est la différence entre un paramètre de route et un paramètre de requête (query parameter) ? Donnez un exemple de chacun.
Un paramètre de route fait partie de l'URL et est défini dans le chemin de la route, par exemple `/books/{book_id}` où `book_id` est un paramètre de route. Un paramètre de requête est ajouté à l'URL après un `?` et est utilisé pour filtrer ou modifier la requête, par exemple `/books?author=John` où `author` est un paramètre de requête.
19. Pourquoi utilise-t-on `PATCH` pour la mise à jour d'un auteur plutôt que `PUT` ?
    `PATCH` est utilisé pour les mises à jour partielles, où seuls certains champs de l'objet sont modifiés, tandis que `PUT` est utilisé pour les mises à jour complètes, où tous les champs de l'objet doivent être fournis.
20. Que fait `payload.model_dump(exclude_unset=True)` dans la route de mise à jour ? Que se passerait-il sans `exclude_unset=True` ?
    `model_dump(exclude_unset=True)` permet de ne retourner que les champs qui ont été modifiés dans le payload, en excluant les champs qui n'ont pas été fournis (non modifiés). Sans `exclude_unset=True`, tous les champs du modèle seraient retournés, même ceux qui n'ont pas été modifiés, ce qui pourrait entraîner des mises à jour non intentionnelles si ces champs sont inclus dans la requête.
21. Pourquoi utilise-t-on `session.get(Author, author_id)` plutôt que `session.execute(select(Author).where(Author.id == author_id))` pour chercher un élément par sa clé primaire ?
    `session.get(Author, author_id)` est une méthode optimisée pour récupérer un objet par sa clé primaire, elle utilise une requête plus simple et plus rapide que `session.execute(select(Author).where(Author.id == author_id))`, qui est plus générique et peut être utilisé pour des requêtes plus complexes. De plus, `session.get()` retourne directement l'objet SQLAlchemy, tandis que `session.execute()` retourne un résultat de requête qui doit être traité pour obtenir l'objet.

### ORM et requêtes

22. Expliquez la différence entre `session.add()` et `session.commit()`. Que se passe-t-il si on appelle `session.add()` sans `session.commit()` ?
    `session.add()` ajoute un objet à la session, ce qui signifie qu'il est marqué pour être inséré dans la base de données lors du prochain commit. Cependant, tant que `session.commit()` n'est pas appelé, les changements ne sont pas persistés dans la base de données. Si on appelle `session.add()` sans `session.commit()`, l'objet sera ajouté à la session mais ne sera pas enregistré dans la base de données, et les changements seront perdus si la session est fermée ou si une exception se produit avant le commit.
23. À quoi sert `session.flush()` ? Dans quels cas l'utilise-t-on ?
    `session.flush()` envoie les changements en attente dans la session à la base de données sans commettre la transaction. Cela permet d'obtenir des valeurs générées par la base de données (comme les clés primaires auto-incrémentées) avant de faire un commit. On l'utilise souvent lorsqu'on a besoin de l'ID d'un objet nouvellement créé pour créer des relations avec d'autres objets avant de faire le commit final.
24. Expliquez la différence entre `joinedload` et `selectinload`. Dans quel cas préfère-t-on l'un à l'autre ?
    `joinedload` effectue une jointure SQL pour charger les relations en une seule requête, ce qui peut être plus efficace pour les relations 1:N ou N:1 où le nombre de résultats est limité. `selectinload` effectue une requête séparée pour charger les relations, ce qui peut être plus efficace pour les relations N:M ou lorsque le nombre de résultats est élevé, car cela évite de dupliquer les données dans la jointure. En général, on préfère `joinedload` pour les relations avec peu de résultats et `selectinload` pour les relations avec beaucoup de résultats.

25. Pourquoi dans FastAPI est-il quasi-obligatoire d'utiliser `selectinload`/`joinedload` lorsqu'on veut retourner des relations ? Que se passe-t-il si on ne le fait pas ?
    Si on ne utilise pas `selectinload` ou `joinedload`, les relations ne seront pas chargées automatiquement lors de la récupération des objets, ce qui peut entraîner des requêtes supplémentaires pour accéder aux données des relations.

26. Quelle est la différence entre ces deux approches ?
    ```python
    # Approche A
    select(Book.id, Book.title, Author.name.label("author_name")).join(Author)

    # Approche B
    select(Book).options(joinedload(Book.author))
    ```
    L'approche A va uniquement sélectionner les champs spécifiés alors que B va sélectionner tout les champs.

## Tâches à réaliser

Les tâches suivantes sont à implémenter dans de nouveaux fichiers ou dans les fichiers existants selon la logique du projet.

### Modèle

#### 1. Table `Person`
Créez un modèle `Person` représentant le propriétaire d'un livre.

- Une personne peut posséder plusieurs livres
- Un livre appartient à une seule personne (relation 1:N)
- Attributs minimum : `id`, `first_name`, `last_name`
- Ajoutez le champ `owner_id` dans le modèle `Book` (avec ou sans `relationship`, à vous de choisir et de justifier)
- Ajoutez des données de test dans `init_db()`

### Routes — Persons

#### 2. Créer une personne
`POST /orm/persons`

- Valider les données avec un schema Pydantic
- Retourner la personne créée

#### 3. Lister les personnes
`GET /orm/persons`

- Retourner la liste de toutes les personnes

#### 4. Personnes avec leurs livres (nom seulement)
`GET /orm/persons-with-books`

- Retourner chaque personne avec la liste des titres de ses livres
- Ne pas retourner l'objet `Book` complet — uniquement le titre (string)
- Choisir la bonne stratégie de chargement et justifier votre choix

### Routes — Livres enrichis

#### 5. Livres avec auteur et éditeur
`GET /orm/books-full`

- Retourner chaque livre avec le nom de l'auteur et le nom de l'éditeur
- Rappel : `publisher_id` existe dans `Book` mais il n'y a pas de `relationship` — la jointure doit être faite manuellement

#### 6. Supprimer un livre
`DELETE /orm/books/{book_id}`

- Retourner `204 No Content` si supprimé
- Retourner `404` si le livre n'existe pas

### Routes — Statistiques

#### 7. Statistiques générales
`GET /orm/stats`

Retourner un objet JSON avec :
- Nombre total de livres
- Nombre total d'auteurs
- Nombre total de tags
- Titre et nombre de pages du livre le plus long
- Moyenne du nombre de pages de tous les livres

#### 8. Personnes avec le nombre de livres
`GET /orm/persons-with-book-count`

- Retourner chaque personne avec le nombre de livres qu'elle possède
- Utiliser une aggregation (`COUNT`) — pas de chargement de la liste des livres

---

## Validation avec PgAdmin

- Créez un fichier `validation.sql` avec des requêtes SQL pour vérifier que les données sont correctement insérées, mises à jour et supprimées dans la base de données
- Exécutez ces requêtes dans PgAdmin pour valider les opérations effectuées par votre API
- Contrôler que les jointures fonctionnent correctement en vérifiant les données retournées par les endpoints qui utilisent des jointures
- Contrôler les valeurs des statistiques retournées par les endpoints de statistiques

## Checklist de validation

- Les nouveaux endpoints apparaissent dans Swagger UI (`/docs`)
- Les validations Pydantic renvoient bien `422` si les données sont invalides
- Les routes `404` fonctionnent correctement
- Le `DELETE` retourne bien `204`
- Les statistiques affichent des valeurs correctes
- Les requêtes avec jointure ne font pas de N+1 (vérifier avec les logs SQL si disponible)
