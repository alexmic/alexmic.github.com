---
layout: post
title: "Delightful testing with pytest and Flask-SQLAlchemy"
tldr: "A succint guide on testing your Flask-SQLAlchemy project with pytest."
published: true
---

If words bore you, you can find the code in this [gist](https://gist.github.com/alexmic/7857543). If you're not familiar with either ```Flask-SQLAlchemy``` or ```pytest```, you can read up on them [here](http://pythonhosted.org/Flask-SQLAlchemy/) and [here](http://pytest.org/latest/contents.html).

### Goals

Before we dive in the code, let's establish two important goals for our test suite:

* **Fast**: Slow tests become a friction point in your development workflow. Database setup and truncating or dropping tables cause delays.
* **Isolated**: Each test should be an introvert, working in their own isolated bubble. You should
never have to think about what other tests have put in the database.

### Test setup

Our example project is structured like follows:

{% highlight python linenos=table %}
project/
  project/
    __init__.py
    models.py
    factory.py
    database.py
    ...
  tests/
    __init__.py
    conftest.py
    test_models.py
    ...
{% endhighlight %}

Expanding a bit on what each module does:

* The ```Flask``` application object is created via a factory method in ```factory.py``` which allows configuration
overriding for different contexts. Apart from configuration, the factory method initializes all plugins with the newly created application.
* The ```Flask-SQLAlchemy``` instance is created in ```database.py``` and exported as ```db```.
* All the necessary test setup takes place in ```pytest``` fixture methods which reside in ```conftest.py```, as shown below.

Note that I am using ```SQLite``` for this example, but the fixtures can be easily adapted to use a different database.

{% highlight python linenos=table %}
import os
import pytest

from project.factory import create_app
from project.database import db as _db


TESTDB = 'test_project.db'
TESTDB_PATH = "/opt/project/data/{}".format(TESTDB)
TEST_DATABASE_URI = 'sqlite:///' + TESTDB_PATH


@pytest.fixture(scope='session')
def app(request):
    """Session-wide test `Flask` application."""
    settings_override = {
        'TESTING': True,
        'SQLALCHEMY_DATABASE_URI': TEST_DATABASE_URI
    }
    app = create_app(__name__, settings_override)

    # Establish an application context before running the tests.
    ctx = app.app_context()
    ctx.push()

    def teardown():
        ctx.pop()

    request.addfinalizer(teardown)
    return app


@pytest.fixture(scope='session')
def db(app, request):
    """Session-wide test database."""
    if os.path.exists(TESTDB_PATH):
        os.unlink(TESTDB_PATH)

    def teardown():
        _db.drop_all()
        os.unlink(TESTDB_PATH)

    _db.app = app
    _db.create_all()

    request.addfinalizer(teardown)
    return _db


@pytest.fixture(scope='function')
def session(db, request):
    """Creates a new database session for a test."""
    connection = db.engine.connect()
    transaction = connection.begin()

    options = dict(bind=connection, binds={})
    session = db.create_scoped_session(options=options)

    db.session = session

    def teardown():
        transaction.rollback()
        connection.close()
        session.remove()

    request.addfinalizer(teardown)
    return session
{% endhighlight %}

So, what happens here? First of all, the ```app``` fixture creates a new application
using the test configuration. In our case, we just switch the database to a test one. It also
establishes a context so all the parts of our ```Flask``` application are properly functioning.

The next fixture layer is the database. The ```db``` fixture creates a new database using the ```create_all()```
method in ```Flask-SQLAlchemy``` and drops all tables after the tests have run. Both the ```db``` and ```app```
fixtures have **session scope**, i.e they will get executed the first time they are requested and then get
cached.

The final layer is the ```session```, which is the fixture we will use in any tests that need database
interaction. This fixture has **function scope** so it runs for every test. It creates a new scoped session and starts a transaction. On teardown, the transaction is rolled back so any changes introduced by the tests are discarded.

#### How does this setup achieve our goal?

As a refresher, our goal is to have fast and isolated tests. By wrapping each test in a transaction we ensure that it will not alter the state of the database since the transaction is rolled back on test teardown. This works for parallelized tests as well.

Apart from isolation, using transactions makes our test suite faster since creating and rolling back transactions is a faster operation than creating and dropping tables between tests (*6.31* seconds versus *3.29* seconds for 29 tests on a Macbook Air using a ```SQLite``` database).

As a plus, if we're running a subset of the tests that requires no database interaction and hence no session fixtures, all the database gruntwork is skipped automatically because of the fixture dependency tree. Nifty.

### Writing tests

{% highlight python linenos=table %}
def test_post_model(session):
    post = Post(title='foo')

    session.add(post)
    session.commit()

    assert post.id > 0
{% endhighlight %}

The above is an example test using the ```session``` fixture. Notice that we are free to commit the session
as we would do normally. This is achieved because the ```session``` "joins" the external transaction created by the connection we explicitly created in the ```session``` fixture, hence only the outermost ```BEGIN```/```COMMIT``` pair has any effect. The documentation at [Joining a Session into an External Transaction](http://docs.sqlalchemy.org/en/latest/orm/session.html#joining-a-session-into-an-external-transaction) has more details on how this works.

### Migrations

As discussed above, the ```db``` fixture creates the database tables using the ```create_all()``` method in ```Flask-SQLAlchemy```. This brings the database to the correct state but it doesn't reflect the way you would create/alter your database schema in production. A popular way of handling migrations with ```SQLAlchemy``` is ```Alembic``` so let's see how we can add that into the mix:

{% highlight python linenos=table %}
from alembic.command import upgrade
from alembic.config import Config

ALEMBIC_CONFIG = '/opt/project/alembic.ini'

def apply_migrations():
    """Applies all alembic migrations."""
    config = Config(ALEMBIC_CONFIG)
    upgrade(config, 'head')
{% endhighlight %}

All we have to do then is replace the ```create_all()``` method call in the ```db``` fixture with the method above. When the ```db``` fixture is first requested, ```alembic``` will apply the migrations in order and bring the database to the state described by your version scripts thus ensuring that they correctly reflect the state of your model layer.

### Caveats

The current version of ```Flask-SQLAlchemy``` on ```PyPI``` (1.0) is outdated so I am using the latest master from Github (2.0-dev). However, the ```SignallingSession``` in ```Flask-SQLAlchemy``` breaks the example at [Joining a Session into an External Transaction](http://docs.sqlalchemy.org/en/latest/orm/session.html#joining-a-session-into-an-external-transaction) so I had to subclass ```SignallingSession``` to make it work. The code for this is included in the [gist](https://gist.github.com/alexmic/7857543) as well.

Thanks to Jocke Ekberg, Faethon Milikouris, George Eracleous and Alex Loizou for reviewing.