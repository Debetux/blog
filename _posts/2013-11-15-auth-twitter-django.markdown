---
layout: post
title:  "Authentification Twitter avec DJango."
date:   2013-11-15 09:00:00
categories: debian
---

Cela est un tout petit tutoriel sur l'intégration de l'OAuth Twitter à Django.
De base, Django nous fournit un module d'authentification. Il est suffisant pour de nombreux cas, et je vais pas me fouler, je vais traduire/résumer ce qu'en dit la documentation de Django [à ce sujet][django-auth-doc].  

### Ce qu'en dit la doc  
#### Aperçu
Django est fourni avec un système d'authentification. Il gère les comptes utilisateurs, groupes, permissions et les sessions basées sur les cookies. Cette section explique comment l'implémentation par défaut marche, et également comment la customiser pour qu'elle convienne à nos projets.

Le système d'authentification de Django prend en charge l'authentification et l'autorisation. Brièvement, l'authentification vérifie sur l'utilisateur est celui qu'il prétend être, et l'autorisation détermine ce que peut faire l'utilisateur authentifié. Le terme d'authentification inclut ces deux tâches.

Le système d'auth consiste en :

- Utilisateurs
- Permissions : détermine si l'utilisateur peut effectuer une certaine tâche, ou pas
- Groupes : un moyen générique d'appliquer des labels et des permissions à plusieurs utilisateurs
- Un système d'hashage de mot de passe configurable
- Outils pour les formulaires et les vues, pour les utilisateurs (ou restreindre du contenu)
- Un système de backend plugable (avec vos propres modules)

Le système d'authentification de Django a pour but d'être très générique, et ne fourni pas toutes les tâches que l'on peut trouver dans d'autres systèmes d'authentification sur le web. Les solutions pour ces problèmes ont été implémentées dans des packages tiers :

- Vérification de la "force" du mot de passe
- Système d'étranglement des tentatives de logins
- Authentification avec des tiers (OAuth par exemple, ce qui est notre cas, héhé)

#### Installation :  
Le module d'authentification est bundlé en tant que Django contrib module dans `django.contrib.auth`.  
Par défaut, la configuration requise est déjà incluse dans le fichier `settings.py` généré par la commande `django-admin.py startproject`. Le système est décomposé en deux items dans la liste des `INSTALLED_APPS` :

- `'django.contrib.auth'` contient le coeur du framework d'authentification, et ces modèles par défaut
- `'django.contrib.contenttypes'` est le Django content type system, qui permet aux permissions d'êtres associées à nos modèles. 

ainsi que de deux items dans notre `MIDDLEWARE_CLASSES` setting :

- `SessionMiddleware` gère les sessions entre requests.
- `AuthenticationMiddleware` associe l'utilisateur avec les requêtes en utilisant les sessions.

Avec ces réglages en place, la commande `manage.py syncdb` va créer les tables nécessaires pour l'authentification, et les permissions pour nos propres modèles. La commande va également demander à créer un superuser.  

## Twitter et Django

Comme vous pouvez le voir, le système d'User de Django va être à la base de notre application. (Moi qui au début, pensait gérer ça moi-même, comme avec PHP, mais non, Django le fait très bien.)

La meilleure solution serait d'ajouter un backend. Il y en a des pré-faits avec le projet [Python-Social-Auth][python-social-auth], mais comme j'ai pas eu plus de temps de me plonger dans la doc... Par contre, j'ai trouvé quelque chose de compréhensible et d'assez simple à installer, et à modifier : [Twython-Django][twython-django].  
Sauf, qu'on va créer tout ça avec nos petites mains pour bien tout comprendre. RDY ? GO !

C'parti les coco, on commence par installer Twython !

{% highlight bash %}
pip install twython
{% endhighlight %}

Lançons notre nouveau projet Django.

{% highlight bash %}
django-admin.py startproject MyProject
cd MyProject
python manage.py startapp twitter_connect
{% endhighlight %}

Fonçons dans le fichier `settings.py` mettre tout cela :

{% highlight python %}
TWITTER_KEY = 'VOTTOKEN'
TWITTER_SECRET = 'VOTTOKENSECRET'

LOGIN_URL='/twitter_connect/login'
LOGOUT_URL='/twitter_connect/logout'
LOGIN_REDIRECT_URL='/'
LOGOUT_REDIRECT_URL='/'

TEMPLATE_DIRS = [os.path.join(BASE_DIR, 'templates')]

INSTALLED_APPS = (
    # ...
    'twitter_connect',
)
{% endhighlight %}

De cette manière, on pourra récupérer les réglages directement depuis le fichier `settings.py`. Vous verrez plus bas. Easy peasy.  
Ensuite, créons un `urls.py` dans notre dossier `twitter_connect/`.

{% highlight python %}
from django.conf.urls import patterns, url

from twitter_connect import views

urlpatterns = patterns('',
	# ex: /polls/
	url(r'^$', views.index, name='index'),

	# First leg of the authentication journey...
    url(r'^login/?$', views.begin_auth, name="twitter_login"),

    # Logout, if need be
    url(r'^logout/?$', views.logout, name="twitter_logout"),  # Calling logout and what not

    # This is where they're redirected to after authorizing - we'll
    # further (silently) redirect them again here after storing tokens and such.
    url(r'^thanks/?$', views.thanks, name="twitter_callback"),

    # An example view using a Twython method with proper OAuth credentials. Clone
    # this view and url definition to get the rest of your desired pages/functionality.
    url(r'^user_timeline/?$', views.user_timeline, name="twitter_timeline"),
)
{% endhighlight %}

Niquel chrome les cocos, mais ça va pas marcher comme ça. Ajoutons ça dans l'`urls.py` du projet lui même :

{% highlight python %}
urlpatterns = patterns('',
	# Examples:
	# url(r'^$', 'CT.views.home', name='home'),
	# url(r'^blog/', include('blog.urls')),

	url(r'^admin/', include(admin.site.urls)),
	url(r'^twitter_connect/', include('twitter_connect.urls')),
)
{% endhighlight %}


Maintenant, le modèle :

{% highlight python %}
from django.contrib.auth.models import User


class TwitterProfile(models.Model):
    """
        An example Profile model that handles storing the oauth_token and
        oauth_secret in relation to a user. Adapt this if you have a current
        setup, there's really nothing special going on here.
    """
    user = models.OneToOneField(User)
    oauth_token = models.CharField(max_length=200)
    oauth_secret = models.CharField(max_length=200)
{% endhighlight %}

voui, voui, on importe le modèle User. De cette manière, lors de l'authentification avec Twitter, nous créeons un nouvel User, et l'auth Django se passera comme sur des roulettes.  
Le `OneToOneField`, c'est, je cite « A one-to-one relationship ». Sauf que les deux objets ne feront qu'un, lorsque l'on voudra les récupérer.

Maintenant, le plus beau. Dans vos views.py, collez y ça :

{% highlight python %}
from django.shortcuts import render

from django.contrib.auth import authenticate, login, logout as django_logout
from django.contrib.auth.models import User
from django.http import HttpResponse
from django.http import HttpResponseRedirect # Pour la redirection
from django.conf import settings # Go chopper les settings, gracieusement logés dans notre settings.py
from django.core.urlresolvers import reverse # Pour favoriser le DRY, voir ici : http://stackoverflow.com/questions/11241668/what-is-reverse-in-django
from django.shortcuts import render_to_response


from twython import Twython

# If you've got your own Profile setup, see the note in the models file
# about adapting this to your own setup.
from twitter_connect.models import TwitterProfile

def index(request):
	return HttpResponse('Hello')

def logout(request, redirect_url=settings.LOGOUT_REDIRECT_URL):
    """
        Nothing hilariously hidden here, logs a user out. Strip this out if your
        application already has hooks to handle this.
    """
    django_logout(request)
    return HttpResponseRedirect(request.build_absolute_uri(redirect_url))


def begin_auth(request):
    """
        The view function that initiates the entire handshake.

        For the most part, this is 100% drag and drop.
    """
    # Instantiate Twython with the first leg of our trip.
    twitter = Twython(settings.TWITTER_KEY, settings.TWITTER_SECRET)

    # Request an authorization url to send the user to...
    callback_url = request.build_absolute_uri(reverse('twitter_connect.views.thanks'))
    auth_props = twitter.get_authentication_tokens(callback_url)

    # Then send them over there, durh.
    request.session['request_token'] = auth_props
    return HttpResponseRedirect(auth_props['auth_url'])


def thanks(request, redirect_url=settings.LOGIN_REDIRECT_URL):
    """A user gets redirected here after hitting Twitter and authorizing your app to use their data.

    This is the view that stores the tokens you want
    for querying data. Pay attention to this.

    """
    # Now that we've got the magic tokens back from Twitter, we need to exchange
    # for permanent ones and store them...
    oauth_token = request.session['request_token']['oauth_token']
    oauth_token_secret = request.session['request_token']['oauth_token_secret']
    twitter = Twython(settings.TWITTER_KEY, settings.TWITTER_SECRET,
                      oauth_token, oauth_token_secret)

    # Retrieve the tokens we want...
    authorized_tokens = twitter.get_authorized_tokens(request.GET['oauth_verifier'])

    # If they already exist, grab them, login and redirect to a page displaying stuff.
    try:
        user = User.objects.get(username=authorized_tokens['screen_name'])
    except User.DoesNotExist:
        # We mock a creation here; no email, password is just the token, etc.
        user = User.objects.create_user(authorized_tokens['screen_name'], "fjdsfn@jfndjfn.com", authorized_tokens['oauth_token_secret'])
        profile = TwitterProfile()
        profile.user = user
        profile.oauth_token = authorized_tokens['oauth_token']
        profile.oauth_secret = authorized_tokens['oauth_token_secret']
        profile.save()

    user = authenticate(
        username=authorized_tokens['screen_name'],
        password=authorized_tokens['oauth_token_secret']
    )
    login(request, user)
    return HttpResponseRedirect(redirect_url)


def user_timeline(request):
    """An example view with Twython/OAuth hooks/calls to fetch data about the user in question."""
    user = request.user.twitterprofile
    twitter = Twython(settings.TWITTER_KEY, settings.TWITTER_SECRET,
                      user.oauth_token, user.oauth_secret)
    user_tweets = twitter.get_home_timeline()
    return render_to_response('tweets.html', {'tweets': user_tweets})
{% endhighlight %}

N'oublions pas, [le template qui va bien][template-file].

Maintenant qu'on a tout, on peut faire un `python manage.py syncdb`, et un `python manage.py runserver` ! Vous pouvez trouver les sources du tutoriel [ici][django-twython-repo].  
Je viens tout juste de commencer avec DJango, donc ce ne sont peut-être pas les meilleures pratiques. Sous peu, je referais le même tutoriel, mais cette fois-ci avec un backend.   

See you soon !

[django-auth-doc]: https://docs.djangoproject.com/en/1.6/topics/auth/
[twython-django]: https://github.com/ryanmcgrath/twython-django
[python-social-auth]: https://github.com/omab/python-social-auth
[django-twython-repo]: https://github.com/Debetux/django-twython
[template-file]: https://github.com/Debetux/django-twython/blob/master/twitter_connect/templates/tweets.html