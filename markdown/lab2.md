# Lab Manual: Object-Relational Data Modeling with Ruby on Rails
-  Course: Database Administration and Management
-  Topic: Object-Relational Data Models
-  Objective: To understand and demonstrate the principles and benefits of object-relational data modeling using Ruby on Rails.
- 
## Part 1: Prerequisites - Installing Ruby on Rails in WSL 
1. Open your Ubuntu WSL terminal
2. Run the following commands:

```bash
sudo apt update
sudo apt upgrade -y
```
3. Install Ruby Version Manager(rbenv)
  - install necessary dependencies for `rbenv`

```bash
sudo apt install -y git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libncurses5-dev libffi-dev libgdbm-dev
```
4. Clone `rbenv` from GitHub:

```bash
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'eval "$(~/.rbenv/bin/rbenv init - bash)"' >> ~/.bashrc
exec bash
```
5. Install ruby-build (an `rbenv` plugin for compiling Ruby versions):

```bash
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
exec bash
```
6. Install Ruby

```bash 
rbenv install 3.4.4
rbenv global 3.4.4
rbenv rehash
ruby -v 
```
7. Install Bundler: Bundler is a dependency manager for Ruby.

```bash
gem install bundler
rbenv rehash
```
8. Install Rails

```bash
gem install rails -v 8.0.2 # Installing Rails 8.0.2 as specified
rbenv rehash
rails -v # Verify Rails installation
```
9. Install required postgres client libraries

```bash
sudo apt install -y libpq-dev`
```

## Part 2: Demonstrating Object-Relational Data Models with Ruby on Rails
In this part, we will build a simple application to manage Books and Authors. This will illustrate how ActiveRecord (Rails' ORM) maps objects to relational database tables and how object-relational concepts like associations simplify data access and manipulation.

**A. Create a New Rails Application**
Navigate to a directory where you want to create your project (e.g., your home directory).
Create a new Rails application named library_app and specify PostgreSQL as the database.

```bash
rails new library_app --database=postgresql
cd library_app
```

This command generates a new Rails application with a PostgreSQL database configuration. By default, Rails 8 will use importmap for JavaScript, which does not require Node.js or Yarn.

**B. Configure Database and Create PostgreSQL User**

Before executing rails db:create, we need to ensure your PostgreSQL setup has a user that matches your WSL Ubuntu username. This allows Rails to connect without explicit password configuration, using peer authentication (common for local development).

- Find your WSL Ubuntu Username:
- Open a new WSL terminal and type:

```bash
whoami
```
(e.g., your_wsl_username)

Access PostgreSQL Command Line:

```bash
 sudo -u postgres psql
 ```

You are now in the PostgreSQL interactive terminal.

- Create a PostgreSQL user matching your WSL username:
Replace your_wsl_username with the actual username you found in step 1.

```sql
CREATE USER your_wsl_username WITH SUPERUSER CREATEDB;
\q
```
This creates a PostgreSQL user with SUPERUSER and CREATEDB privileges, which is sufficient for local development.

Configure config/database.yml (if necessary):
Open the config/database.yml file in your favorite editor (e.g., nano config/database.yml or open in VS Code if you have code . set up in WSL).
By default, Rails will try to connect using the current OS user, which matches the PostgreSQL user we just created. You typically won't need to change the username or password under the development and test sections if your PostgreSQL user matches your WSL username and peer authentication is in use. It should look something like this (ensure username is commented out or matches your WSL username):

```yaml
YAML

# config/database.yml
# ...
development:
  adapter: postgresql
  encoding: unicode
  # For details on connection string, see Rails documentation
  # disable_prepared_statements: true
  database: library_app_development
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  # username: your_wsl_username # Often not needed if your PG user matches OS user
  # password: your_password # Often not needed if your PG user matches OS user
  host: localhost
  port: 5432
# ...
```

Create the Rails databases:

```bash
rails db:create
```

This command executes the database creation process. You should see output indicating the development and test databases were created successfully.

**C. Generate Models (Tables and Classes)**

We'll create two models: Author and Book. A Book will belong to an Author.

- Generate Author Model:

```bash
rails generate model Author name:string bio:text
```
This command generates:

- app/models/author.rb: The Ruby class for Author.
- db/migrate/<timestamp>_create_authors.rb: A database migration file to create the authors table.
- Generate Book Model:

Here, we'll establish the relationship. A Book will belong_to an Author. This means the books table will have an author_id foreign key.

```bash
rails generate model Book title:string isbn:string published_at:date author:references
```
- The author:references part is crucial. It tells Rails to:

- Add an author_id column (of type bigint) to the books table.
- Add a foreign key constraint to author_id referencing the authors table.
- Automatically add belongs_to :author to the Book model and has_many :books to the Author model (we'll see this in the next step).
- Run Migrations:

```bash
rails db:migrate
```

This command executes the migration files, creating the authors and books tables in your PostgreSQL database.

Inspect the Database: You can now connect to your PostgreSQL database (e.g., using psql library_app_development) and run \d authors; and \d books; to see the table schemas. Notice the author_id column in books and the foreign key constraint.

**D. Define Associations in Models**

While author:references helps, we explicitly define the associations in the model files for clarity and to enable all ORM features.

Open app/models/author.rb:

```ruby
class Author < ApplicationRecord
  has_many :books, dependent: :destroy # Added dependent: :destroy for cascading delete
end
```
- `has_many :books` signifies that an `Author` can have multiple `Books`. The `dependent: :destroy` option ensures that when an `author` is deleted, all their associated `books` are also deleted.


Open `app/models/book.rb:`

```ruby
class Book < ApplicationRecord
  belongs_to :author
end
```
belongs_to :author signifies that a Book belongs to one Author.

- **Strength of ORM:** This is a core strength! Instead of manually joining tables, you define relationships at the object level, and the ORM handles the underlying SQL.

**E. Interact with Data via Rails Console (Object-Relational in Action)**

Now, let's use the Rails console to demonstrate how object-relational mapping simplifies data manipulation and to observe the SQL generated.

- Start the Rails console with SQL logging enabled:

```bash
rails console
ActiveRecord::Base.logger = Logger.new(STDOUT) # This will log SQL queries to the console
```
- Create Authors:

```ruby
author1 = Author.create(name: "J.K. Rowling", bio: "British author, creator of the Harry Potter fantasy series.")
author2 = Author.create(name: "Stephen King", bio: "American author of horror, suspense, science fiction, and fantasy.")
author3 = Author.create(name: "Agatha Christie", bio: "English crime novelist, short story writer, and playwright.")
```

- **Observe SQL:** You will see `INSERT INTO "authors"` statements being logged.

- Create Books and Associate them with Authors:

```ruby
# Books for J.K. Rowling
book1 = author1.books.create(title: "Harry Potter and the Philosopher's Stone", isbn: "978-0747532699", published_at: "1997-06-26")
book2 = author1.books.create(title: "Harry Potter and the Chamber of Secrets", isbn: "978-0747538493", published_at: "1998-07-02")

# Books for Stephen King
book3 = author2.books.create(title: "It", isbn: "978-0451169518", published_at: "1986-09-15")
book4 = author2.books.create(title: "The Shining", isbn: "978-0345806789", published_at: "1977-01-28")

# Books for Agatha Christie
book5 = author3.books.create(title: "And Then There Were None", isbn: "978-0007136834", published_at: "1939-11-06")
```
- **Observe SQL:** Notice the `INSERT INTO "books"` statements. Pay attention to how the `author_id` is automatically populated by Rails.

- **Retrieve Data (Object-Oriented Way):**

- Find an author by ID:

```ruby
jk_rowling = Author.find(author1.id)
puts jk_rowling.name
```
- **Observe SQL:** An `SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2` query is generated.

- Access an author's books directly:

```ruby
jk_rowling.books.each do |book|
  puts "  - #{book.title}"
end
```

- **Observe SQL:** Rails executes a `SELECT "books".* FROM "books" WHERE "books"."author_id" = $1` query. You don't write the `JOIN` or `WHERE` clause explicitly.

-Find a book by title and access its author:

```ruby
shining_book = Book.find_by(title: "The Shining")
puts "Book: #{shining_book.title}, Author: #{shining_book.author.name}"
```
- **Observe SQL:** When you call `shining_book.author.name`, Rails performs another `SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2` query on the `authors` table using the `author_id` from the `shining_book` object. This lazy loading is another ORM benefit.

- Add a new book to an existing author:

```ruby
new_hp_book = jk_rowling.books.create(title: "Harry Potter and the Half-Blood Prince", isbn: "978-0747581086", published_at: "2005-07-16")
jk_rowling.books.count # Should show 3 now
```

- **Observe SQL:** Another INSERT INTO "books" will be logged.

- Find all books by a specific author (using where):

```ruby
stephen_king_books = Book.where(author: author2) # You can pass the author object directly!
stephen_king_books.each do |book|
  puts book.title
end
```

- **Observe SQL:** This translates to `SELECT "books".* FROM "books" WHERE "books"."author_id" = $1;`.

- Query across associations (using joins):

```ruby
# Find books written by authors whose name starts with 'J'
books_by_j_authors = Book.joins(:author).where("authors.name LIKE ?", "J%")
books_by_j_authors.each do |book|
  puts "#{book.title} (by #{book.author.name})"
end
```
- **Observe SQL:** Rails generates a `SELECT` statement with an `INNER JOIN` between `books` and `authors` tables, e.g., `SELECT "books".* FROM "books" INNER JOIN "authors" ON "authors"."id" = "books"."author_id" WHERE (authors.name LIKE 'J%').`

- Update a book's information:

```ruby
shining_book.update(published_at: "1977-01-01") # Just an example
shining_book.reload.published_at # Verify the update
```
- **Observe SQL:** This performs an `UPDATE "books"` SQL statement.

- Delete an author and observe cascading effects:

```ruby
jk_rowling = Author.find_by(name: "J.K. Rowling")
jk_rowling.destroy # This will also destroy associated books because of `dependent: :destroy`
Book.where(author: jk_rowling).count # Should be 0
```
- **Observe SQL:** You will see a `DELETE FROM "books"` for each of J.K. Rowling's books, followed by a `DELETE FROM "authors"` for J.K. Rowling herself. This demonstrates how object-level configurations `(dependent: :destroy)` translate into multiple SQL operations orchestrated by the ORM for data integrity.

## Key Takeaways for Students

- **Object-Relational Mapping (ORM):** Understand that an ORM (like ActiveRecord in Rails) acts as a bridge between your object-oriented programming language (Ruby) and a relational database (PostgreSQL).
- **Objects vs. Tables:** You primarily interact with Ruby objects (Author, Book) rather than directly writing SQL queries.
- **Associations:** ORMs simplify defining and working with relationships between data (e.g., has_many, belongs_to). These relationships are reflected in your database schema (foreign keys) and handled by the ORM at the object level.
- **Productivity:** ORMs significantly boost developer productivity by abstracting away SQL complexities, handling data type conversions, and providing a consistent API for database interactions.
- **Maintainability:** Changes to your data model often require only changes to your Ruby models, rather than widespread SQL modifications.
- **Observing SQL:** It's crucial to understand the SQL queries an ORM generates. This helps in debugging, optimizing performance, and truly grasping the underlying relational model even while working with objects.
- **Database Agnostic (to an extent):** While we used PostgreSQL, the Ruby code for Author and Book models would remain largely the same if you switched to MySQL or SQLite, thanks to the ORM's abstraction layer.
- **Trade-offs:** Discuss that while ORMs are powerful, they can introduce a "performance penalty" if not used carefully (e.g., N+1 queries). Understanding the SQL being generated is still important for optimization.