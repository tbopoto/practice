#instagram clone

git remote add origin https://github.com/tbopoto/practice.git
git branch -M main
git push -u origin main
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import desc

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///instagram.db'
db = SQLAlchemy(app)

# Define the User model
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password = db.Column(db.String(120), nullable=False)
    posts = db.relationship('Post', backref='author', lazy=True)

    def __init__(self, username, email, password):
        self.username = username
        self.email = email
        self.password = password

# Define the Post model
class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    caption = db.Column(db.String(200))
    image_url = db.Column(db.String(200))
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)

    def __init__(self, caption, image_url, user_id):
        self.caption = caption
        self.image_url = image_url
        self.user_id = user_id

# Create the database tables
db.create_all()

# User registration endpoint
@app.route('/register', methods=['POST'])
def register_user():
    data = request.json
    username = data.get('username')
    email = data.get('email')
    password = data.get('password')

    if User.query.filter_by(username=username).first():
        return jsonify({"error": "Username already exists"}), 400

    new_user = User(username=username, email=email, password=password)
    db.session.add(new_user)
    db.session.commit()

    return jsonify({"message": "User registered successfully"})

# Post creation endpoint
@app.route('/post', methods=['POST'])
def create_post():
    data = request.json
    username = data.get('username')
    password = data.get('password')
    caption = data.get('caption')
    image_url = data.get('image_url')

    user = User.query.filter_by(username=username, password=password).first()
    if not user:
        return jsonify({"error": "Invalid credentials"}), 401

    new_post = Post(caption=caption, image_url=image_url, user_id=user.id)
    db.session.add(new_post)
    db.session.commit()

    return jsonify({"message": "Post created successfully"})

# User feed endpoint
@app.route('/feed/<username>', methods=['GET'])
def get_user_feed(username):
    user = User.query.filter_by(username=username).first()
    if not user:
        return jsonify({"error": "User not found"}), 404

    user_feed = Post.query.filter_by(user_id=user.id).order_by(desc(Post.id)).all()
    feed_data = [{"caption": post.caption, "image_url": post.image_url} for post in user_feed]

    return jsonify(feed_data)

if __name__ == '__main__':
    app.run(debug=True)
#ENDDDDDD
