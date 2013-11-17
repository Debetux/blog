---
layout: post
title:  "[MEMO-PHP] Stockage des mots de passe de manière sécurisée."
date:   2013-05-02 09:00:00
categories: password codeigniter php
---

Aujourd'hui, je bosse sous CodeIgniter, et je suis en train de créer la partie inscription/authentification utilisateur.  
Aucune librairie ne fait ce que je veux. J'ai déjà une base de site web, avec une table bien définie pour les utilisateurs. Je ne veux pas la modifier (ou légèrement si il le faut). Seulement, la plupart des libraires sont des mastodontes, comme [IonAuth][ion-auth]. [IonAuth][ion-auth], est, de mon point de vue, une vraie usine à gaz. 3000 mille lignes de code pour un modèle. Certes, elle fait plein de choses, et sûrement bien ! Mais elle utilise son propre schéma de base de données. Je ne vois pas parcourir 3000 lignes pour remplacer des variables. J'aime aussi avoir la main sur ce qu'il se passe, et le code de [Ionauth][ion-auth] est bien éparpillé, et confus (de mon point de vue).

Puis j'ai trouvé [Phpass][phpass]. C'est exactement ce dont j'ai besoin, et la libraire est plutôt simple d'utilisation. Elle sert à créer un hash d'un mot de passe (qu'on ne peut retrouver à partir du hash). Je me pencherais sur son fonctionnement plus tard. Mais je suis tellement heureux d'avoir enfin trouvé une solution !

Voici un exemple de modèle :

{% highlight php %}
<?php
class Auth_model extends CI_Model {

    protected $table = 'users';

    public function __construct()
    {
        parent::__construct();
        $this->load->database();
        
    }

    public function add_user($username, $email, $first_name, $last_name, $password, $acl_id = '1'){
        # On charge la librairie.
        $this->load->library('PasswordHash');
        # On initialise l'objet.
        $pwdHasher = new PasswordHash(8, FALSE);
        # Pour hasher un mot de passe, rien de plus simple :
        $hash = $pwdHasher->HashPassword($password);

        # On a plus qu'a tout insérer dans la base de donnée.
        $this->db->set('username',  $username)
                ->set('email',   $email)
                ->set('first_name', $first_name)
                ->set('last_name', $last_name)
                ->set('password', $hash)
                ->set('acl_id', $acl_id);

        $this->db->set('created_time', 'NOW()', false)->set('updated_time', 'NOW()', false);
         
        # Si le hash fait moins de 10 caractères, il y a un problème.
        if(strlen($hash) > 10)	return $this->db->insert($this->table);
        else return false;
    }

    public function verify_user($username, $password){
        # On charge la librairie.
        $this->load->library('PasswordHash');
        # On initialise l'objet.
        $pwdHasher = new PasswordHash(8, FALSE);

        # On va chercher le hash du mot de passe stocké dans la base de donnée.
        $row = $this->db->select('password')->from($this->table)->where(array('username' => $username))->get()->result();
        
        $password_stored = $row[0]->password;

        # La magie opère !
        return $pwdHasher->CheckPassword($password, $password_stored);
    }
}
{% endhighlight %}

Pour ceux utilisant du PHP pur, retenez juste ces deux lignes pour la création du hash :
{% highlight php %}
<?php  # On initialise l'objet.
$pwdHasher = new PasswordHash(8, FALSE);
# Pour hasher un mot de passe, rien de plus simple :
$hash = $pwdHasher->HashPassword($password);
{% endhighlight %}

Puis ça pour vérifier que le mot de passe est le bon :
{% highlight php %}
<?php # On initialise l'objet.
$pwdHasher = new PasswordHash(8, FALSE);
$password_hash = $row[0]->password;
# La magie opère !
return $pwdHasher->CheckPassword($password, $password_hash);
{% endhighlight %}

Cette page est plus un mémo qu'autre chose. Si vous voulez plus de précisions, vous pouvez demander à [DuckDuckGo][ddg], ou bien laisser un commentaire. Dans un prochain billet, je donnerais des astuces pour nginx ainsi que comment sécuriser son serveur. 

[ion-auth]: https://github.com/benedmunds/CodeIgniter-Ion-Auth
[phpass]: http://www.openwall.com/phpass/
[ddg]: https://duckduckgo.com/
