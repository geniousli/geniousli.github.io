---
title: Phoenix framework 1.3 context
date: 2017-08-07
tags: phoenix, elixir, desigin
---

Design
--------

### 思考设计

  context 是专门用来 组织、暴露 相关的功能的module。比如我们每次调用Elixir标准库 Logger.info, 其实是在接触不同的context， 在内部， Elixir Logger是由 诸如 Logger.Config. Logger.Backends 的module组成的， 但是我们从来不会直接跟这些module交互， 我们使用Logger context， 因为他组织并且暴露接口。
  Phoenix 组织目录类似于其他的Elixir Project， 我们拆分代码到context中， 一个context 将会组织相关的功能代码， 比如post，comment， 经常封装诸如 验证、数据存取的功能。应用context, 我们拆分系统到 容易管理、互相独立的 组成部分中。

### 建立Account context
  * user 是 系统中广泛接触的，以至于需要思考设计 接口。 我们的目标是设计一个 Account API 处理创建、更新、删除user、包括用户验证。 我们以基本的功能开始，然后逐步添加功能。
  * Phoenix 包含了 phx.gen.html, phx.gen.json, phoenix.gen.context 脚手架来支持 拆分 应用功能到 contex中， 这些脚手架有力的推动了在 应用变大 的时候，朝着正确的方向上前进。
  * 为了使用 context 脚手架， 我们需要想出一个module名字，来组织相关的功能。在Ecto Guide中，我们使用Changesets 和Repo 来验证，存储user， 但是当应用变大的时候，我们不会使用这些在应用中。事实上，我们从来不会想， user应该存在于应用的什么地方，退回重新思考一下 应用中个个组成部分的不同。 在用户 需要账户登录，验证，和注册的情况下，Account context 是一个最好的地方来存放相关的功能。

  ```elixir
    mix phx.gen.html Accounts User user users name:string username:string:unique
    resources "/users", UserController
    mix ecto.migrate

    scopes "/" HelloWeb do
      pipe_through :browser

      get "/", PageController, :index
      resources "/users", UserController
    end
  ```
  * Phoenix 生成的web相关的文件， 在 lib/hello_web/中， context文件在 lib/hello/accounts 中， 注意 这个区别， 我们使用Accounts module来管理account相关的功能， Accounts.User struct 是Ecto 来转换、验证的user的模型，

### starting with Generators
  * phx.gen.html  脚手架 生成了一个开箱能用的 创建，更新，删除用户。离真正的app很远。但是脚手架 是第一个也是最重要的 开始真正构建真是的功能的 学习工具和开始的地方。 脚手架不能解决所有问题，但是依然可以教你朝着一个正确的方向 来思考设计你的应用 。

  ``` elixir
  defmodule HelloWeb.UserController do
    use HelloWeb, :controller
    alias Hello.Accounts

    def index(conn, _params) do
      users = Accounts.list_users()
      render(conn, "index.html", users: users)
    end

    def new(conn, _params) do
      changeset = Accounts.change_user(%Hello.Accounts.User{})
      render(conn, "new.html", changeset: changeset)
    end

    def create(conn, %{"user" => user_params}) do
      case Accounts.create_user(user_params) do
        {:ok, user} ->
          conn
          |> put_flash(:info, "user create success")
          |> redirect(to: user_path(conn, :show, user))
        {:error, %Ecto.Changeset{} = changeset} ->
          render(conn, "new.html", changeset: changeset)
      end
    end
  end
  ```
  * 我们注意的是， Controller如何调用Accounts context，我们可以看到index action，通过Accounts.list_users获取一系列用户，通过Accounts.create_user调用 实现创建用户，我们并不知道 Accounts，创建、获取的实现方法，然而这才是重要的， Phoenix Controller， 就是web interface， 她并不关系 关于如何获取用户， 创建的细节。我们只关心　告诉我们的应用为我们执行一些任务。我们将业务逻辑存储引擎通过web　层分离开来，如果我们转移到一个全文索引来代替sql查询，我们的Controller不需要改动。
  * 在这个create示例中，当我们成功的创建了一个用户，我们调用Phoenix.Controller.push_flash　来展示成功的消息　然后重定向到user_path　，　如果失败了，　我们渲染"new.html"模板，并且传递Ecto.changeset　来传递错误信息
  * 下面，我们会更深入的挖掘Account　context　
  * 在这个帐号管理系统中，我们可能需要处理用户登录凭证，用户喜好，和密, 码重置等。如果我们深入到list_users 函数中，我们可以看到获取用户的细节，Repo.all(User), 意图是对caller隐藏从Postgresql获取的细节，　这是Phoenix　生成器一个常见的主题。Phoenix会驱动我们去思考，系统中不同的组成部分，然后包装这些不同的部分到Well-named　的module中、function　让我们代码的意图更加清晰，同时屏蔽细节。

### In-context relationship

　*  我们基础的用户功能实现的非常好，让我们补充实现用户登录，我们不会实现一个完整的认证系统，而只是给我们一个系统成长的好的开始。许多的验证系统的解决方案是　凭证跟用户是一对一的。这样经常造成问题，比如支持不同的登录方式，例如social的给你路， email邮箱登录。将会造成主要代码逻辑改变，我们要建立一个凭证、用户一对一，但是容易支持其他功能。
  * 就当前，用户凭证之包含email信息，我们地一个任务是决定凭证放在哪里。我们有Accounts context，用户凭证适合放在这里。Phoenix也足够聪明在已经存在的context生成代码

  > mix phx.gen.context Accounts Credenttial credentials email:string:unique user_id:references:users
  我们使用phx.gen.context 区别与 phx.gen.html 除了不会生成 html，因为我们已经有了Controller
  Template
  我们可以从输出看到Phoenix为我们的Accounts.Credential 生成了 accounts/credential.ex , 还有migration
  在执行迁移之前，我们需要修改生成的迁移文件。我们需要删除用户的凭证，当删除用户的时候，执行下面的改动


  ```elixir
  def change do
      create table(:credentials) do
        add :email, :string
        add :user_id, references(:users, on_delete: :nothing)
        add :user_id, references(:users, on_delete: :delete_all), null: false
        timestamps()
      end

      create uniqu_index(:credentials, [:email])
      create index(:credentials, [:user_id])
  end
  ```

  我们将:on_delete 从nothing 改变为:delete_all，这就会生成外键约束，约束会删除所有的用户凭证，当用户删除的时候, 我们设定null: false 来阻止 创建没有关联用户的凭证创建。应用数据库约束，我们在数据层面限制data， 而不是在应用层面。
  在我们与web 层面交互之前，我们需要 让我们的context 知道关联的user 和credentials ， 编辑lib/accoutns/user.ex

  ```elixir

  alias Hello.Accounts.{User, Credential}

  schema "users" do
    field :name, :string
    field :username, :string
    has_one :credential, Credential
    timestamps()
  end
  ```

  我们使用Ecto.Schema's has_one macro来让Ecto知道如何关联User and Credential， 同样的编辑accounts/credential.ex
  ```elixir
  alias Hello.Accounts.{Credential, User}
  schema "credential" do
    field :email, :string
    field :user_id, :id
    belongs_to :user, User
    timestamps()
  end
  ```
  ```elixir
  def list_users do
    User
    |> Repo.all()
    |> Repo.preload(:credenttial)
  end

  def get_user!(id) do
    User
    |> Repo.get!(id)
    |> Repo.preload(:credential)
  end
  ```
  我们重写了list_user 和get_user以便于主动加载credential 关联关系，无论什么时候我们获取用户，Repo.preload 函数获取schema的关联关系。当我们操作一个集合，比如list_users， Ecto可以高效的预加载关联关系在一个sql中， 这允许我们直接调用credients，而不需要额外的 查询。
  下面我们在页面中加入credential

  ``` html
  + <div class="form-group">
  +   <%= inputs_for f, :credential, fn cf -> %>
  +     <%= label cf, :email, class: "control-label" %>
  +     <%= text_input cf, :email, class: "form-control" %>
  +     <%= error_tag cf, :email %>
  +   <% end %>
  + </div>

  <div class="form-group">
    <%= submit "Submit", class: "btn btn-primary" %>
  </div>
  ```

  使用 Phoenix.HTML's inputs_for 函数来添加一个关联的嵌套的fields， 在嵌套的inputs中，我们渲染了 credfential‘s 的email error_tag 等。
  展示用户的user 的email地址在用户的show页面中。添加下面的代码到show.html.eex中

  ```html
  + <li>
  +   <strong>Email:</strong>
  +   <%= @user.credential.email %>
  + </li>
  </ul>
  ```

  现在如果我们访问users/new 我们会看到email的输入框，如果保存，你会发现email自大un被护士了，没有验证告诉你字段没有被保存。
  我们是哟个Ecto's belongs_to has_one 关联关系来贯穿起来，为了关联用户输入到数据库，我们需要在changeset中处理，改动Accounts context中的create_user , update_user 函数。

  ```elixir
  def update_user(%User{} = user, attrs) do
    user
    |> User.changeset(attrs)
    |> Ecto.Changeset.cast_assoc(:credential, with: &Credential.changset/2)
    |> Repo.update()
  end

  def create_user(attrs \\ %{}) do
    %User{}
    |> User.changset(attrs)
    |> Ecto.Changeset.cast_assoc(:credential, with： &Credential.changset/2)
    |> Repo.insert()
  end
  ```
  Ecto's cast_assoc 告诉changset如何转换用户的输入到schema 关系中。我们使用with告诉changset使用Credential.changeset函数来转换用户输入，这样，Credential.changset 中所有的验证会应用到 User中。
  最后，我们可以测试，会见到Email的验证。

### Adding Account functions

  如你所见， context是一个包粗一组相关函数的module， Phoenix 生成一般的函数， 例如list_users, upate_users, 但是他们只提供一个基础的，为了使用一些真是的功能来扩展我们的Account context， 让我们修复我们应用中的明显的问题， -我们可以创建带有凭证的用户，但是却无法使用凭证登录，建立一个负载的完整的用户验证系统超出了本书的范围，让我们着手建立一个基于email的登录页面来跟踪用户的session， 这会让我们聚焦在Accounts context上， 并且给你一个建立完成的认证系统上一个好的开始。
  首先，开始想名字，为了使用email地址来验证用户，我们需要一个方法来查找用户和验证凭证是否正确，我们只暴露一个函数来完成这件事情。

  ```elixir
    > user = Accounts.authenticate_by_email_password(email, password)
  ```

  这看起来很好， 一个描述的名字暴露了我们代码的意图。这一功能使它清楚它充当什么目的，同时允许调用者仍然一无所知的内部细节。 lib/hello/accounts/accounts.ex

  ```elixir
   def authenticate_by_email_password(email, password) do
       query =
           from u in User,
           inner_join：c in assoc(u, :credential)，
           where: c.email == ^email
       case Repo.one(query) do
           %User{} = user -> {:ok, user}
           nil -> {:error, :unauthorized}
       end
   end
   ```
  我们定义的authenticate_by_email_password 函数， 我们忽略掉了password 字段，你可以使用FIX ME 留在以后构建。我们需要的函数是使用验证凭证 来寻找到user， 返回包含%Accounts.User{} and :ok tuple， 或者一个{:error, :unauthorized} 来让调用者知道他们的验证是错误的。
  现在我们使用context来验证用户，我们添加一个login page 在我们的web，首先创建一个新的controller 在 lib/hello_web/controllers/session_controller.ex

  ```elixir
  defmodule HelloWeb.SessionController do
      use HelloWeb, :controller
      alias Hello.Accounts

      def new(conn, _) do
          render(conn, "new.html")
      end

      def create(conn,k %{"user" => %{"email" => email, "password" => password}}) do
          case Accounts.authenticate_by_email_password(email, password) do
              {:ok, user} ->
                  conn
                  |> put_flash(:info, "welcome back")
                  |> put_session(:user_id, user.id)
                  |> configure_session(renew: true)
                  |> redirect(to: "/")
              {:error, :unauthorized} ->
                  conn
                  |> put_flalsh(:error, "bad email/password combination")
                  |> redirect(to: session_path(conn, :new))
          end
      end
  end
  ```
  我们定义了一个SessionController来处理用户signing and out，new action 只是简单的渲染 "new session"表单， 表单简单的post 到create action， 我们使用模式匹配然后调用 Accounts.authenticate_by_email_password， 如果成功的话，我们使用Plug.Conn.put_session 来将验证过的用户ID放到 session中， 然后跳转到home page， 我们需要调用 configure_session(conn, renew: true) 在重定向之前，来避免session fixation 个哦高年级， 如果验证错误， 我们添加一个falsh error， 并重定向 四个in-in， 我们添加一个delete action 简单的调用Plug.Conn.configure_session 来删除掉session并重定向到home page
  lib/hello_web/router.ex

  ```elixir
  scope "/" HelloWeb do
      pipe_through :brower

      get "/" PageController, :index
      resources "/users", UserController
      resources "/sessions", SessionController, only: [:new, :create, :delete], singleton: true
  end
  ```
  我们使用resources 来生成 一系列的routes 在"/sessions"下，使用:only 来限制我们的route 生成。因为我么只需要支持 :new, :create, :delete action， 我们添加singleton: true 选项，来定义restful routes， 但是不需要一个resource ID 在URL中。 我们不需要一个ID在URL中，因为我们的action 总是知道current_user， 这个ID总是在session中，在我们完成controller之前， 我们添加一个验证的plug 在route中来限制用户访问， lib/hello_web/router.ex

  ```elixir
  def authenticate_user(conn, _) do
      case get_session(conn, :user_id) do
          nil ->
              conn
              |> Phoenix.Controller.put_flash(:error, ”login please)
              |> Phoenix.Controller.redirect(to: "/")
              |> halt()
          user_id ->
              assign(conn, :current_user, Hello.Accounts.get_user!(user_id))
      end
  end
  ```
  我们定义了authenticate_user plug 在路由中，只是简单的使用get_sessioin 来检查session中的:user_id， 如果我们找到了， 以为着user 已经验证过了，我们就调用Accounts.get_user!，并在当前的connection assigns中赋值:current_user， 如果没有找到，我们就flash 一个错误信息， 重定向到 home page上， 我们使用Halt来组织后面的代码被进一步调用，
  最后，我们需要SessionView来渲染模板， lib/hello_web/views/session_view.ex
  ```elixir
  defmodule HelloWeb.SessioinView do
      use HelloWeb, :view
  end
  ```

#### cross-context dependencies

  我们已经开始了 用户的 帐号与登录凭证 功能，我们开始开发我们应用的重要功能， 管理页面内容，我们开发一个支持用户创建、编辑 网站页面的cms内容。我们可以拓展用户系统。如果我们回国头来想想应用的独立性，这个并不合适，一个用户系统并不应该关心cms系统，Account context 的职责是管理用户与凭证，不处理任何的页面内容。这时候需要一个独立的context来进行管理。CMS
  我们创建一个CMS context 来处理cms 的职责，在我们写代码之前，可以想想CMS 的功能有这些：
  1. Page 的create、 update
  2. Page 属于创建的用户
  3. 用户的信息应该展示在页面中
  从描述中，我们清楚的知道需要一个Page resource来存储page， 那author信息怎么办？ 我们拓展现有的Account.User来包含， Role等信息， 这违反了我们定义的context原则，我们定义了User context为什么User context 需要注意到author 的信息呢？
  存在用户的应用中，会自然的成为一个严重的User驱动应用。但是，我们的应用总是为用户使用设计的。与其扩展Account.User结构来追踪平台每个字段的变化，不如，让拥有职责功能的module来负责。在我们的应用中，我们创建CMS.Author 结构来管理Author中管理CMS 的字段信息。

  ```elixir
  mix phx.gen.html CMS Page pages title:string body:text views:integer --web CMS
  mix ecto.migrate
  ```
  ```elixir
  scope "/cms", HelloWeb.CMS as: :cms do
      pipe_through [:bowser, :authenticate_user]
      resources "/pages", PageController
  end
  ```
  我们添加了 authenticate_user 来要求在CMS scope中的路由都需要用户登录。
  创建Author
  ```elixir
  mix phx.gen.context CMS Author authors bio:text role:string gere:string user_id:reference:users:unique
  mix ecto.migrate
  ```
  我们添加了 bio， role，genre， user_id在author中。这样可以保证，CMS API不会因为User context的改动而改动。
  ```elixir
  def change do
      create table(:authors) do
          add :bio, :text
          add :role, :string
          add :genre, :string
          add :user_id, references(:users, on_delete: :nothing)
          add :user_id, references(:users, on_delete: :delete_all), null: false

          timestamps()
      end

      create unique_index(:authors, [:user_id])
  end
  ```
  我们添加了delete_all，来限制当用户删除的时候，我们不要依靠应用代码来清理CMS author代码。
  开始之前， 我们需要一个新的数据迁移，我们有一个author表，我们需要关联pages authors， 我们添加个author_id字段到pages，
  ```elixir
  mix ecto.gen.migration add_author_id_to_pages

  def change do
      alter table(:pages) do
          add :author_id, references(:authors, on_delete: :delete_all), null: false
      end
      create index(:pages, [:author_id])
  end

  mix ecto.migrate
  ```
#### Cross-context data

  依赖在系统中总是不可避免的，但我们进最大的努力来限制他们，当有可能减少以来的时候，我们已经完美的使用context 来区分应用的不同部分，但是我们依然需要处理依赖。
  Author 来负责CMS中的author， CMS context 会对Account context存在一个数据上的依赖。我们有两个选择，1：在Account context中暴露API 来允许我们在CMS中方便的获取User。 2：通过数据库的join操作来获取依赖部分的数据，都是可能的选项，但是join data 对于一个硬的数据依赖，对于大应用来说是很好的，如果你决定拆分context 分散到不同应用，数据中，你依然享受隔离的好处，因为你的开放的接口保持不变。
  /lib/hello/cms/page.ex

  ```elixir
  - alias Hello.CMS.Page
  + alis Hello.CMS.{Page, Author}

  schema "pages" do
      field :body, :string
      field :title, :string
      field :views, :integer
      belongs_to :author, Author
      timestamps()
  end
  ```
  添加了author page, 之间的关联关系
  ```elixir
  - alias Hello.CMS.Author
  + alias Hello.CMS.{Author, Page}

  schema "authors" do
      field :bio, :string
      field :genre, :string
      field :role, :string
      field :user_id, :id


      has_many :pages, Page
      belongs_to :user, Hello.Accounts.User
      timestamps()
  end
  ```
  下面我们来改变 CMS context需要一个author当更新、创建page的时候，
  ```elixir
  alias Hello.CMS.{Page, Author}
  alias Hello.Accounts

  def list_pages do
      Page
      |> Repo.all()
      |> Repo.preload(author: [user: :credential])

  end

  def get_page!(id) do
       Page
       |> Repo.get!(id)
       |> Repo.preload(author: [user: :credential])
  end

  def get_author!(id) do
      Author
      |> Repo.get!(id)
      |> Repo.preload(user: :credential)
  end
  ```

  我们在list_pages, get_page中，preload 关联的author， user，credential 从数据库中，
  我们来处理 数据的存储， 在创建、编辑page的时候，保存author， 编辑文件, lib/hello/cms/cms.ex
  ```elixir
  def create_page(%Author{} = author, attrs \\ %{}) do
      %Page{}
      |> Page.changeset(attrs)
      |> Ecto.Changeset.put_change(:author_id, author.id)
      |> Repo.insert()
  end

  def ensure_author_exists(%Accounts.User{} = user) do
     %Author{user_id: user.id}
     |> Ecto.Changeset.change()
     |> Ecto.Changeset.unique_constraint(:user_id)
     |> Repo.insert()
     |> handle_existing_author()
  end

  defp handle_existing_author({:ok, author}), do: author

  defp handle_existing_author({:error, changeset}) do
     Repo.get_by!(Author, user_id: changeset.data.user_id)
  end
  ```

  lib/helo_web/controllers/cms/page_controller.ex
  ```elixir
      plug :require_existing_author
      plug :authorize_page when action in [:edit, :update, :delete]

      def require_existing_author(conn, _) do
          author = CMS.ensure_author_exists(conn.assigns.current_user)
          asign(conn, :current_author, author)
      end

      defp authorize_page(conn, _) do
          page = CMS.get_page!(conn.params["id"])

          if conn.assigns.current_author.id == page.author_id do
              assign(conn, :page, page)
          else
              conn
              |> put_flash(:error, "you can't modify this page")
              |> redirect(to: cms_page_path(conn, :index))
              |> halt()
          end
      end
  ```
  我们在 CMS.PageController中添加了 两个Plug, 地一个plug， :require_existing_author, 在每个action都需要执行， 函数 调用CMS.ensure_author_exists 并传递current_user过去，当找到 或者创建author，我们使用Plug.Conn.assign 来将 current_author 传递到下去。
  第二步，我们使用:authorized_page来过滤特定的action， 函数 地一个从connection params获取page，，然后跟current_author 进行验证， 如果current_author 的ID 跟Page的author_id一样，可以通过验证， 如果不一样，我们Plug.Conn.halt 来阻止， action的进一步执行。
  ```elixir
   def create(conn, %{"page" => page_params}) do
  +   case CMS.create_page(conn.assigns.current_author, page_params) do
      {:ok, page} ->
        conn
        |> put_flash(:info, "Page created successfully.")
        |> redirect(to: cms_page_path(conn, :show, page))
      {:error, %Ecto.Changeset{} = changeset} ->
        render(conn, "new.html", changeset: changeset)
    end
  end

  def update(conn, %{"page" => page_params}) do
  +   case CMS.update_page(conn.assigns.page, page_params) do
      {:ok, page} ->
        conn
        |> put_flash(:info, "Page updated successfully.")
        |> redirect(to: cms_page_path(conn, :show, page))
      {:error, %Ecto.Changeset{} = changeset} ->
        render(conn, "edit.html", changeset: changeset)
    end
  end

  def delete(conn, _) do
  +   {:ok, _page} = CMS.delete_page(conn.assigns.page)

    conn
    |> put_flash(:info, "Page deleted successfully.")
    |> redirect(to: cms_page_path(conn, :index))
  end     

 ```
   我们改变了create 的写法，从connection assign中获取current_author, authenticate_user 会存放进去， 然后我们传递current_author 到 CMS.create_page 中 来关联到page。
   在web view中展示 author's name
   lib/hello_web/views/cms/page_view.ex
   ```elixir
   defmodule HelloWeb.CMS.PageView do
       use HelloWeb, :view
       alias Hello.CMS

       def author_name(%CMS.Page{author: author}) do
        author.user.name
       end
   end
   ```
   完成！我现在有两个独立的分别负担 user context 和内容管理
