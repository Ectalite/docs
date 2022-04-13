# Démarrage rapide

---

GoAdmin est facile à utiliser dans divers frameworks web grâce à divers adaptateurs. Les frameworks web actuellement supportés sont :

- [gin](http://github.com/gin-gonic/gin)
- [beego](https://github.com/astaxie/beego)
- [fasthttp](https://github.com/valyala/fasthttp)
- [buffalo](https://github.com/gobuffalo/buffalo)
- [echo](https://github.com/labstack/echo)
- [gorilla/mux](http://github.com/gorilla/mux)
- [iris](https://github.com/kataras/iris)
- [chi](https://github.com/go-chi/chi)
- [gf](https://github.com/gogf/gf)

<br>

Vous pouvez opter pour le framework que votre projet utilise. S'il n'y a pas de framework qui vous convient, n'hésitez pas à nous faire part d'un problème (issue) ou d'un PR (Pull request) !

Prenons le framework gin comme exemple pour démontrer le processus de création.

## main.go

Tout d'abord, créez un nouveau fichier `main.go` dans le dossier de votre projet avec le contenu suivant:

```go
package main

import (
	_ "github.com/GoAdminGroup/go-admin/adapter/gin" // Importez l'adaptateur, il doit être importé. S'il n'est pas importé, vous devez le définir vous-même.
	_ "github.com/GoAdminGroup/themes/adminlte" // Importer le thème
	_ "github.com/GoAdminGroup/go-admin/modules/db/drivers/mysql"
	"github.com/GoAdminGroup/go-admin/engine"
	"github.com/GoAdminGroup/go-admin/examples/datamodel"
	"github.com/GoAdminGroup/go-admin/modules/config"
	"github.com/GoAdminGroup/go-admin/modules/language"
	"github.com/GoAdminGroup/go-admin/plugins/admin"
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()

	// Instancie un objet moteur GoAdmin.
	eng := engine.Default()

	// Configuration globale de GoAdmin, peut aussi être écrite en json pour être importée.
	cfg := config.Config{
		Databases: []config.Database{
			{
				Host:         "127.0.0.1",
				Port:         "3306",
				User:         "root",
				Pwd:          "root",
				Name:         "godmin",
				MaxIdleCon:   50,
				MaxOpenCon:   150,
				Driver:       "mysql",
			},
		},
		UrlPrefix: "admin", // Le préfixe url du site web.
		// Le Store doit être défini et garanti comme ayant un accès en écriture, sinon les nouveaux utilisateurs administrateurs ne pourront pas être ajoutés.
		Store: config.Store{
			Path:   "./uploads",
			Prefix: "uploads",
		},
		Language: language.FR,
	}

	// Ajoutez la configuration et les plugins, utilisez la méthode Use pour accéder au framework web.
	_ = eng.AddConfig(cfg).
		// About Generators，see: https://github.com/GoAdminGroup/go-admin/blob/master/examples/datamodel/tables.go
		AddGenerators(datamodel.Generators).
		Use(r)

	_ = r.Run(":9033")
}
```

Veuillez prêter attention au code et aux commentaires ci-dessus, les étapes correspondantes sont ajoutées aux commentaires, il est simple à utiliser. Voici le résumé en 5 étapes :

- Importez l'adaptateur, le thème et le pilote sql.
- Définir les éléments de configuration globale
- Initialiser le plugin
- Configurer les plugins et les paramètres
- accéder au framework web

<br>

Ensuite, exécutez `go run main.go` pour exécuter le code et accéder à: [http://localhost:9033/admin/login](http://localhost:9033/admin/login) <br>
<br>
compte utilisateur par défaut: admin<br>
mot de passe par défaut: admin

plus d'exemple de framework web : [https://github.com/GoAdminGroup/go-admin/tree/master/examples](https://github.com/GoAdminGroup/go-admin/tree/master/examples)

## Ajoutez vos propres tables métiers pour la gestion

See：<br><br>
1 [Comment utiliser les plugins](plugins/plugins)<br>
2 [Comment utiliser le plugin Admin](plugins/admin)

## Description de l'élément de configuration globale

[https://github.com/GoAdminGroup/go-admin/blob/master/modules/config/config.go](https://github.com/GoAdminGroup/go-admin/blob/master/modules/config/config.go)

```go
package config

import (
	"html/template"
)

// La base de données est un type de configuration de connexion de base de données.
// Parce qu'il y a une petite différence entre les différents pilotes de base de données.
// La configuration comporte plusieurs options mais peut ne pas être utilisée.
// Par exemple, le pilote sqlite n'utilise que l'option FILE
// qui peut être ignorée lorsque le pilote est mysql.

type Database struct {
	Host         string
	Port         string
	User         string
	Pwd          string
	Name         string
	MaxIdleCon   int
	MaxOpenCon   int
	Driver       string
	File         string
}

// Configuration de la base de données
// qui est un map dans lequel la clé (key) est le nom de la connexion à la base de données et
// La valeur (value) est la configuration de données correspondante.
// La clé (key) est la base de données par défaut, mais aussi
// la base de données utilisée par le framework, et vous pouvez configurer plusieurs
// bases de données à utiliser par vos tables métier pour gérer différentes bases de données.

type DatabaseList map[string]Database

// Store est la configuration du stockage de fichiers. Path est le chemin du stockage local.
// et prefix est le préfixe de l'url utilisé pour y accéder.
type Store struct {
	Path   string
	Prefix string
}


// Le type Config est la configuration globale de goAdmin. Elle sera
// initialisée dans le moteur.
type Config struct {
	// Un map supporte la connexion à plusieurs bases de données. Le premier
	// élément de Databases est la connexion par défaut. Voir le
	// fichier connection.go.
	Databases DatabaseList `json:"database"`

	// Le domaine du cookie utilisé dans les modules auth. cf.
	// le fichier session.go.
	Domain string `json:"domain"`

	// Sert à définir la langue à utiliser qui sera affichée dans
	// l'interface.
	Language string `json:"language"`

	// Le préfixe global de l'URL.
	UrlPrefix string `json:"prefix"`

	// Nom du thème du template
	Theme string `json:"theme"`

	// Le chemin dans lequel les fichiers seront stockés.
	Store Store `json:"store"`

	// Titre de la page Web.
	Title string `json:"title"`

	// Le logo est le texte supérieur de la barre latérale.
	Logo template.HTML `json:"logo"`

	// Le mini-logo est le texte supérieur de la barre latérale
	// lorsqu'elle est pliée.
	MiniLogo template.HTML `json:"mini_logo"`

	// L'url à rediriger après la connexion
	IndexUrl string `json:"index"`

	// Mode débogage
	Debug bool `json:"debug"`

	// Env est l'environnement, qui pourrait être local, test, prod.
	Env string

	// Chemin du journal d'information
	InfoLogPath string `json:"info_log"`

	// Chemin du journal d'erreurs
	ErrorLogPath string `json:"error_log"`

	// Chemin d'accès aux journaux
	AccessLogPath string `json:"access_log"`

	// Interrupteur d'enregistrement de l'opérateur Sql
	SqlLog bool `json:"sql_log"`

	AccessLogOff bool
	InfoLogOff   bool
	ErrorLogOff  bool

	// Schéma de couleurs
	ColorScheme string `json:"color_scheme"`

	// Durée de vie de la session, l'unité est la seconde.
	SessionLifeTime int `json:"session_life_time"`

	// Lien Cdn des actifs
	AssetUrl string `json:"asset_url"`

	// Moteur de téléversement des fichiers, par défaut "local".
	FileUploadEngine FileUploadEngine `json:"file_upload_engine"`

	// html personnalisé dans l'en-tête (head) de la balise.
	CustomHeadHtml template.HTML `json:"custom_head_html"`

	// html personnalisé après le body.
	CustomFootHtml template.HTML `json:"custom_foot_html"`

	// Titre de la page de connexion
	LoginTitle string `json:"login_title"`

	// Logo de la page de connexion
	LoginLogo template.HTML `json:"login_logo"`
}

```

<br>

> Le français n'est pas ma langue dominante. Si vous trouvez une quelconque faute de frappe ou de traduction, vous pouvez nous proposer votre aide pour la traduction sur [github here](https://github.com/GoAdminGroup/docs). Je vous en serai très reconnaissant.