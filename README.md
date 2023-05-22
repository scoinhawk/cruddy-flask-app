# cruddy-flask-app
# cruddy-flask-app. This Flask App was started with Bing-Chat 
# 
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///example.db'
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

    def __repr__(self):
        return '<User %r>' % self.name

@app.route('/users', methods=['GET', 'POST'])
def users():
    if request.method == 'POST':
        name = request.json['name']
        email = request.json['email']
        user = User(name=name, email=email)
        db.session.add(user)
        db.session.commit()
        return jsonify({'message': 'User created!'})
    else:
        users = User.query.all()
        output = []
        for user in users:
            user_data = {}
            user_data['id'] = user.id
            user_data['name'] = user.name
            user_data['email'] = user.email
            output.append(user_data)
        return jsonify({'users': output})

@app.route('/users/<id>', methods=['GET', 'PUT', 'DELETE'])
def user(id):
    user = User.query.get_or_404(id)

    if request.method == 'GET':
        user_data = {}
        user_data['id'] = user.id
        user_data['name'] = user.name
        user_data['email'] = user.email
        return jsonify({'user': user_data})

    elif request.method == 'PUT':
        name = request.json['name']
        email = request.json['email']
        user.name = name
        user.email = email
        db.session.commit()
        return jsonify({'message': 'User updated!'})

    elif request.method == 'DELETE':
        db.session.delete(user)
        db.session.commit()
        return jsonify({'message': 'User deleted!'})

if __name__ == '__main__':
    app.run(debug=True)
