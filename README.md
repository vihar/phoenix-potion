# Phoenix Potion
Phoenix Framework blogging application.

To start your Phoenix app:

  * Install dependencies with `mix deps.get`
  * Create and migrate your database with `mix ecto.create && mix ecto.migrate`
  * Install Node.js dependencies with `npm install`
  * Start Phoenix endpoint with `mix phoenix.server`

Now you can visit [`localhost:4000`](http://localhost:4000) from your browser.

Ready to run in production? Please [check our deployment guides](http://www.phoenixframework.org/docs/deployment).

## Learn more

  * Official website: http://www.phoenixframework.org/
  * Guides: http://phoenixframework.org/docs/overview
  * Docs: https://hexdocs.pm/phoenix
  * Mailing list: http://groups.google.com/group/phoenix-talk
  * Source: https://github.com/phoenixframework/phoenix

# Guide

## Elixir Basics

brew install elixir

`iex`, `elixir` and `elixirc` are three executables.

`iex` -> Interactive Elixir

Open your terminal or command prompt and try few operations and functions.

### Basic operations and functions in elixir with numbers
```
Interactive Elixir (1.4.2) - press Ctrl+C to exit (type h() ENTER for help)

iex(1)> 2+3
5
iex(2)> 4*23
92
```
div(10,5)

rem(3,4)

rem(2,1)

trunc(4,3)

### Printing Hello World

IO.puts(“Hello World”)

Notice the IO.puts/1 function returns the atom :ok as result after printing.

Use the pin operator ^ when you want to pattern match against an existing variable’s value rather than rebinding the variable.

Use the _ operator to bind a value for temporary clause.


## Phoenix Commands

```
$ mix phoenix.gen.html → which creates: model, view, controllers, repository, templates, tests
```

```
$ mix phoenix.gen.channel → which creates: channel and tests
```

```
$ mix phoenix.gen.json → for API, which creates: model, view, controllers, repository, tests
```

```
$ mix phoenix.gen.model → which creates: model and repository
```





For our blog application we have to create the post controller so Post will be the model and posts will be the router

```
$ mix phoenix.gen.html Post posts title:string body:text
```

This commands creates a model,and a controller where all the CRUD operations will automatically generated.

```
resources "/posts", PostController
```
Add this in web/router.ex file its defining the URL pattren to access the controller.

```elixir
scope "/", BlogPhoenix do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    resources "/posts", PostController
  end
```

Create and migrate the database by using the following commands

```
$ mix ecto.create
$ mix ecto.migrate
```

## To see our routing list we can use:

```
$ mix phoenix.routes
```

To add comments to your blog posts let's create another model which extends from our post model to do that we have give foreign key as post_id:references:posts

Use the following command to create the model
```
$ mix phoenix.gen.model Comment comments name:string content:text post_id:references:posts
```

To fetch the following post ids into the comment model
Write these in the web/models/comment.ex

```elixir
 @required_fields ~w(name content post_id)
 @optional_fields ~w()
```
The other side of our associations is that our post has many comments so we go into: web/models/post.ex and add:

```
 has_many :comments,
 ```

```elixir
 schema "posts" do
    field :title, :string
    field :body, :string

    has_many :comments, BlogPhoenix.Comment

    timestamps()
```

Migrate so that comments table will be created
```
$ mix ecto.migrate
```

In Post Controller add the comment urls to do that use the following code

```elixir
resources "/posts", PostController do
  post "/comment", PostController, :add_comment
end
```

## Add Comment Function in post controller

```elixir
def add_comment(conn, %{"comment" => comment_params, "post_id" => post_id}) do
  changeset = Comment.changeset(%Comment{}, Map.put(comment_params, "post_id", post_id))
  post = Post |> Repo.get(post_id) |> Repo.preload([:comments])

  if changeset.valid? do
    Repo.insert(changeset)

    conn
    |> put_flash(:info, "Comment added.")
    |> redirect(to: post_path(conn, :show, post))
  else
    render(conn, "show.html", post: post, changeset: changeset)
  end
end
```

In web/controllers/post_controller.ex inside the index function, we need to use the count_comments function from above:
```elixir
def index(conn, _params) do
  posts = Post
  |> Post.count_comments
  |> Repo.all
  render(conn, "index.html", posts: posts)
end
```

Now create a comment from template:
 web/templates/post/comment_form.html.eex

 ```elixir
<%= form_for @changeset, @action, fn f -> %>
  <%= if f.errors != [] do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below:</p>
      <ul>
        <%= for {attr, message} <- f.errors do %>
          <li><%= humanize(attr) %> <%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="form-group">
    <label>Name</label>
    <%= text_input f, :name, class: "form-control" %>
  </div>

  <div class="form-group">
    <label>Content</label>
    <%= textarea f, :content, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= submit "Add comment", class: "btn btn-primary" %>
  </div>
<% end %>
```

Now create a template to show all these values
web/templates/post/comments.html.eex

```elixir
<h3> Comments: </h3>
<table class="table">
  <thead>
    <tr>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
<%= for comment <- @post.comments do %>
    <tr>
      <td><%= comment.name %></td>
      <td><%= comment.content %></td>
    </tr>
<% end %>
  </tbody>
</table>
```
Let's see how our blog post application works:
`http://localhost:4000/posts`
