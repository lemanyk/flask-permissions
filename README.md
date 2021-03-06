# Flask-Permissions

[![Build Status](https://travis-ci.org/raddevon/flask-permissions.png?branch=master)](https://travis-ci.org/raddevon/flask-permissions)

Flask-Permissions is a simple Flask permissions extension that works with [Flask-SQLAlchemy](https://github.com/mitsuhiko/flask-sqlalchemy). It also plays nicely with [Flask-Login](https://github.com/maxcountryman/flask-login) although that isn't a strict requirement.

## Installation

Installs quickly and easily using PIP:

    pip install Flask-Permissions

## Getting Started

1. Import Flask, Flask-SQLAlchemy, and, if you want, Flask-Login.

        from flask import Flask
        from flask.ext.login import LoginManager, current_user
        from flask.ext.sqlalchemy import SQLAlchemy

2. Import the `Permissions` object.

        from flask.ext.permissions.core import Permissions

3. Instantiate the `Permissions` object passing in your Flask app, SQLAlchemy database, and a proxy for the current user.

        perms = Permissions(app, db, current_user)

4. Sub-class the Flask-Permissions UserMixin. Call the UserMixin's `__init__` in your own `__init__`.

        from app import db
        from flask.ext.permissions.models import UserMixin


        class User(UserMixin):
            # Add whatever fields you need for your user class here.

            def __init__(self, email, password, roles=None):
                UserMixin.__init__(self, roles)

5. Put those decorators to work! Decorate any of your views with the `user_is` or `user_has` decorators from `flask.ext.permissions.decorators` to limit access.

        from flask.ext.permissions.decorators import user_is, user_has

    `@user_is` decorator:

        @app.route('/admin', methods=['GET', 'POST'])
        @user_is('admin')
        def admin():
            return render_template('admin.html')

    `@user_has` decorator:

        @app.route('/delete-users', methods=['GET', 'POST'])
        @user_has('delete_user')
        def delete_users():
            return render_template('delete-users.html')

## Example Implementation

This is ripped almost directly from a project I'm working on that implements Flask-Permissions. Be sure to check out the code comments for help with what does what.

#### \__init__.py

    # Import Flask, Flask-SQLAlchemy, and maybe Flask-Login
    from flask import Flask
    from flask.ext.login import LoginManager, current_user
    from flask.ext.sqlalchemy import SQLAlchemy

    # Import the Permissions object
    from flask.ext.permissions.core import Permissions

    # Here, you'll initialize your app with Flask and your database with
    # Flask-SQLAlchemy. It might look something like this:
    db = SQLAlchemy()

    app = Flask(__name__)
    app.config.from_object('config')

    db.init_app(app)
    with app.test_request_context():
        db.create_all()

    # If you're using Flask-Login, this would be a good time to set that up.
    login_manager = LoginManager()
    login_manager.init_app(app)

    # Now, initialize a Permissions object. I've assigned it to a variable here,
    # but you don't have to do so.
    perms = Permissions(app, db, current_user)

#### models.py

    # Import your database
    from app import db
    # I'm using these handy functions for my user's password. Flask is dependent
    # on Werkzeug, so you'll have access to these too.
    from werkzeug import generate_password_hash, check_password_hash
    # Import the mixin
    from flask.ext.permissions.models import UserMixin


    class User(UserMixin):
        # Add whatever fields you need for your user class. Here, I've added
        # an email field and a password field
        email = db.Column(db.String(120), unique=True)
        pwdhash = db.Column(db.String(100))

        def __init__(self, email, password, roles=None):
            self.email = email.lower()
            self.set_password(password)
            # Be sure to call the UserMixin's constructor in your class constructor
            UserMixin.__init__(self, roles)

        def set_password(self, password):
            self.pwdhash = generate_password_hash(password)

        def check_password(self, password):
            return check_password_hash(self.pwdhash, password)

        def __str__(self):
            return self.email

#### views.py

    # Import the decorators
    from flask.ext.permissions.decorators import user_is, user_has

    # Set up your route and decorate it
    @app.route('/admin', methods=['GET', 'POST'])
    # Pass the name of the role you want to test for to the decorator
    @user_is('admin')
    def admin():
        return render_template('admin.html')

    # Here's an example of user_has
    @app.route('/delete-users', methods=['GET', 'POST'])
    # Pass the name of the ability you want to test for to the decorator
    @user_has('delete_user')
    def delete_users():
        return render_template('delete-users.html')

## License

This extension is available under the MIT license. See the LICENSE file for more details.

## Thank You

I hope you enjoy this project. I built Flask-Permissions because I couldn't find a simple permissions system for Flask. This does everything I need, and I feel the implementation is very easy to understand.

I'd love to hear your comments either on Twitter ([@raddevon](http://twitter.com/raddevon/)) or via email ([devon@raddevon.com](mailto:devon@raddevon.com)). I welcome pull requests, but I'm just one guy (and still a relatively new coder) so please try to be patient with me. If this project helps you, consider leaving me a [gittip](https://www.gittip.com/raddevon/).
