Mga Komponent
#############

Ang mga Komponent ay ang mga package sa lohika na ibinabahagi sa pagitan ng mga controller.
Ang CakePHP ay may isang napakagandang set ng pangunahing mga komponent na pwede mong gamitin para tulungan ka sa 
iba-ibang mga karaniwang mga gawain. Maaari ka ring lumikha ng sarili mong mga komponent. Kung makikita mo 
sa sarili na gusto mong i-copy at paste ang mga bagay sa pagitan ng mga controller, dapat mong 
isaalang-alang ang paglikha ng iyong sariling komponent na naglalaman ng functionality. Ang paglilikha ng mga 
komponento ay nagpapanitling malinis sa controller ng code at nagpapahintulot sa iyo upang gamitin ang code sa pagitan 
ng iba't ibang mga controller.

Para sa karagdagang impormasyon sa mga component na kasama sa CakePHP, tingnan ang 
kabanata para sa bawat komponent:

.. toctree::
    :maxdepth: 1

    /controllers/components/authentication
    /controllers/components/cookie
    /controllers/components/csrf
    /controllers/components/flash
    /controllers/components/security
    /controllers/components/pagination
    /controllers/components/request-handling

.. _configuring-components:

Pag-configure ng mga Komponent
==============================

Marami sa pangunahing mga komponent ay nangangailangan ng kumpigurasyon. Ilang mga halimbawa sa mga komponent 
na nangangailangan ng kumpigurasyon ay :doc:`/controllers/components/authentication` at
:doc:`/controllers/components/cookie`.  Ang kumpigurasyon para sa mga komponent na ito,
at para sa mga komponent sa pangkalahatan, ay karaniwang natapos sa pamamagitan ng ``loadComponent()`` sa iyong 
mga Controller na ``initialize()`` na pamamaraan o sa pamamagitan sa ``$components`` na array::

    class PostsController extends AppController
    {
        public function initialize()
        {
            parent::initialize();
            $this->loadComponent('Auth', [
                'authorize' => 'Controller',
                'loginAction' => ['controller' => 'Users', 'action' => 'login']
            ]);
            $this->loadComponent('Cookie', ['expires' => '1 day']);
        }

    }

Maaari mong ma-configure ang mga komponent sa runtime gamit ang ``config()`` na pamamaraan. Madalas,
ito ay natapos sa iyong controller na ``beforeFilter()`` na pamamaraan. Ang itaas ay maaari 
ding ipahayag bilang::

    public function beforeFilter(Event $event)
    {
        $this->Auth->config('authorize', ['controller']);
        $this->Auth->config('loginAction', ['controller' => 'Users', 'action' => 'login']);

        $this->Cookie->config('name', 'CookieMonster');
    }

Tulad ng mga helper, ang mga komponent ay nagpapatupad ng ``config()`` na pamamaraan na ginagamit upang kumuha at 
magtakda ng anumang kumpigurasyon ng datos para sa komponent::

    // Basahin ang config na datos.
    $this->Auth->config('loginAction');

    // Itakda ang config
    $this->Csrf->config('cookieName', 'token');

Tulad ng mga helper, ang mga komponent ay awtomatikong mag-merge sa kanilang ``$_defaultConfig``
na katangian na may kumpigurasyon ng constructor para malikha ang ``$_config`` na katangian
na kung saan ay ma-access sa pamamagitan ng ``config()``.

Pag-alias ng mga Komponent
--------------------------

Isang karaniwang setting na gagamitin ay ang ``className`` na opsyon, na kung saan ay nagpapahintulot sa iyo na
i-alias ang mga komponent. Ang katangiang ito ay kapaki-pakinabang kapag kailangan mong
palitan ang ``$this->Auth`` o ibang karaniwan na Component na nagrereperensiya na may pasadyang
pagpapatupad::

    // src/Controller/PostsController.php
    class PostsController extends AppController
    {
        public function initialize()
        {
            $this->loadComponent('Auth', [
                'className' => 'MyAuth'
            ]);
        }
    }

    // src/Controller/Component/MyAuthComponent.php
    use Cake\Controller\Component\AuthComponent;

    class MyAuthComponent extends AuthComponent
    {
        // Idagdag ang iyong code para i-override ang core AuthComponent
    }

Ang nasa itaas ay *alias* ``MyAuthComponent`` sa ``$this->Auth`` sa iyong
mga controller.

.. note::

    Pag-alias ng komponent ay nagpapalit sa instance na kung saan ang komponent ay ginamit,
    kasama sa loob ng ibang mga Component.

Pag-load ng mga Component na nauna
----------------------------------

Maaaring hindi mo kailangan ang lahat ng iyong mga komponent na magagamit sa bawat controller
na aksyon. Sa mga sitwasyon na tulad nito ay maaari kang maka-load ng isang komponent sa runtime gamit ang
``loadComponent()`` na pamamaraan sa iyong controller::

    // Sa isang controller na aksyon
    $this->loadComponent('OneTimer');
    $time = $this->OneTimer->getTime();

.. note::

    Isaisip na ang mga komponentong naunang naka-load ay hindi makakalimot sa mga natawag na mga callback.
    Kung ikaw ay umaasa sa ``beforeFilter`` o ``startup``
    na mga callback na tinatawag, maaari mong kailangan na tumawag sa kanila ng mano-mano depende kung kailan
    ka nag-load ng iyong komponent.

Paggamit ng mga Komponent
=========================

Sa sandaling iyon ay kasama ang ilang mga komponent sa iyong controller, ang paggamit ng mga ito ay masyadong
simple. Sa bawat komponentong nagamit mo ay nakalantad bilang isang katangian ng iyong controller. Kung
ikaw ay naka-load na nang :php:class:`Cake\\Controller\\Component\\FlashComponent`
sa iyong controller, maaari mong ma-access kung gusto mo::

    class PostsController extends AppController
    {
        public function initialize()
        {
            parent::initialize();
            $this->loadComponent('Flash');
        }

        public function delete()
        {
            if ($this->Post->delete($this->request->getData('Post.id')) {
                $this->Flash->success('Post deleted.');
                return $this->redirect(['action' => 'index']);
            }
        }

.. note::

    Dahil pareho na mga Modelo at mga Komponent ay idinadagdag sa mga Controllers bilang
    mga property na ibinabahagi nila sa parehong 'namespace'. Tiyakin na hindi magbigay ng parehong pangalan 
    sa komponento at sa isang modelo..

.. _creating-a-component:

Paglikha ng Komponent
=====================

Ipagpalagay na ang ating aplikasyon ay nangangailangan na gawin ang kumplikadong mathematical na operasyon sa
maraming ibang mga parte sa aplikasyon.  Maaari tayong lumikha ng isang komponent para ibahay
ang nakabahaging logic na ito para gamitin sa maraming ibang mga controller.

Ang unang hakbang ay ang paglikha ng isang bagong file at class. Lumikha ng file sa
**src/Controller/Component/MathComponent.php**. Ang pangunahing istraktura para sa
component ay mukhang ganito ang anyo::

    namespace App\Controller\Component;

    use Cake\Controller\Component;

    class MathComponent extends Component
    {
        public function doComplexOperation($amount1, $amount2)
        {
            return $amount1 + $amount2;
        }
    }

.. note::

    Lahat ng mga component ay dapat mag-extend sa :php:class:`Cake\\Controller\\Component`. Ang pakakalimot sa paggawa 
    nito ay mag-trigger sa isang exception.

Isasama ang iyong Component sa iyong mga Controllers
----------------------------------------------------

Sa sandaling ang ating komponent ay natapos, maaari nating gamitin ito sa mga controller ng ating aplikasyon
sa pamamagitan ng pag-load nito sa panahon na ang controller ay naka ``initialize()`` na pamamaraan.
Sa sandaling na-load, ang controller ay ibibigay ang isang bagong attribute na nakapangalan sa 
component, kung saan maaaring maka-access ng instance nito::

    // Sa isang controller
    // Gumawa ng bagong component na magagamit sa $this->Math,
    // pati na rin ang pamantayang $this->Csrf
    public function initialize()
    {
        parent::initialize();
        $this->loadComponent('Math');
        $this->loadComponent('Csrf');
    }

Kapag kabilang ang mga Component sa isang Controller ay maaari ka ring mag-declare ng
set ng mga parameter na naipapasa sa constructor ng Component
Itong mga parameter ay maaaring makahawak sa pamamagitan
ng Component::

    // Sa iyong controller.
    public function initialize()
    {
        parent::initialize();
        $this->loadComponent('Math', [
            'precision' => 2,
            'randomGenerator' => 'srand'
        ]);
        $this->loadComponent('Csrf');
    }

Ang itaas ay papasa ng array na naglalaman ng katumpakan at randomGenerator sa
``MathComponent::initialize()`` na nasa ``$config`` na parameter.

Paggamit ng Ibang mga Component sa iyong Komponent
--------------------------------------------------

Minsan isa sa iyong mga komponent ay maaaring kailangan na gumamit ng ibang komponent.
Sa kasong ito ay maaari kang magsama ng ibang mga komponent na eksaktong parehong 
paraan na sinama mo sila sa mga controller - gamit ang ``$components`` var::

    // src/Controller/Component/CustomComponent.php
    namespace App\Controller\Component;

    use Cake\Controller\Component;

    class CustomComponent extends Component
    {
        // The other component your component uses
        public $components = ['Existing'];

        // Execute any other additional setup for your component.
        public function initialize(array $config)
        {
            $this->Existing->foo();
        }

        public function bar()
        {
            // ...
        }
    }

    // src/Controller/Component/ExistingComponent.php
    namespace App\Controller\Component;

    use Cake\Controller\Component;

    class ExistingComponent extends Component
    {

        public function foo()
        {
            // ...
        }
    }

.. note::

    Sa kaibahan sa isang komponent na sinama sa isang controller
    walang mga callback na ma-trigger sa isang component sa component.

Pag-access ng Component ng Controller
-------------------------------------

Mula sa loob ng Komponent maaari kang mag-access sa kasalukuyang controller sa pamamagitan ng
registry::

    $controller = $this->_registry->getController();

Maaari mong i-access ang sa anumang callback na pamamaraan mula sa event
object::

    $controller = $event->getSubject();

Component ng mga Callback
=========================

Ang mga komponento ay nag-aalok rin ng iilang mga hiling sa cycle ng buhay na mga callback na nagpapahintulot sa kanila upang
dagdagan ang hiling ng cycle.

.. php:method:: beforeFilter(Event $event)

    Is called before the controller's
    beforeFilter method, but *after* the controller's initialize() method.

.. php:method:: startup(Event $event)

    Is called after the controller's beforeFilter
    method but before the controller executes the current action
    handler.

.. php:method:: beforeRender(Event $event)

    Is called after the controller executes the requested action's logic,
    but before the controller renders views and layout.

.. php:method:: shutdown(Event $event)

    Is called before output is sent to the browser.

.. php:method:: beforeRedirect(Event $event, $url, Response $response)

    Is invoked when the controller's redirect
    method is called but before any further action. If this method
    returns ``false`` the controller will not continue on to redirect the
    request. The $url, and $response parameters allow you to inspect and modify
    the location or any other headers in the response.

.. meta::
    :title lang=en: Components
    :keywords lang=en: array controller,core libraries,authentication request,array name,access control lists,public components,controller code,core components,cookiemonster,login cookie,configuration settings,functionality,logic,sessions,cakephp,doc
