from flask import Flask, render_template, request, redirect, url_for
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
import uuid

app = Flask(__name__)

# Configure the database
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://root:Donotforget25@localhost/itemdb'


db = SQLAlchemy(app)
migrate = Migrate(app, db)


class Item(db.Model):
    __tablename__ = 'items'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(255), nullable=False)
    description = db.Column(db.Text, nullable=True)

    # Define a foreign key relationship to Image
    image_id = db.Column(db.Integer, db.ForeignKey('images.id'))
    image = db.relationship('Image', back_populates='items')


class Image(db.Model):
    __tablename__ = 'images'

    id = db.Column(db.Integer, primary_key=True)
    filename = db.Column(db.String(36), unique=True, nullable=False)

    # Back reference to Item
    items = db.relationship('Item', back_populates='image')

    def __init__(self):
        self.filename = str(uuid.uuid4())


with app.app_context():
    db.create_all()


@app.route('/')
def browse_items():
    items = Item.query.all()
    return render_template('browse.html', items=items)


# Example of adding a new item
@app.route('/add', methods=['GET', 'POST'])
def add_item():
    if request.method == 'POST':
        name = request.form['name']
        description = request.form['description']

        # Create a new Image instance and associate it with the item
        new_image = Image()
        db.session.add(new_image)
        db.session.commit()

        new_item = Item(name=name, description=description, image=new_image)
        db.session.add(new_item)
        db.session.commit()

        return redirect(url_for('browse_items'))

    return render_template('add.html')


@app.route('/sort/<sort_by>')
def sort_items(sort_by):
    if sort_by == 'id':
        items = Item.query.order_by(Item.id).all()
    elif sort_by == 'name':
        items = Item.query.order_by(Item.name).all()
    else:
        # Handle invalid sorting parameter
        return "Invalid sorting parameter"
    return render_template('browse.html', items=items)


@app.route('/search', methods=['POST'])
def search_item():
    keyword = request.form['keyword']
    # Check if the input can be converted to an integer (ID search)
    try:
        item_id = int(keyword)
        item = Item.query.get(item_id)
        if item:
            items = [item]
        else:
            items = []
    except ValueError:
        # If not a valid integer, search by keyword
        items = Item.query.filter(Item.name.ilike(f"%{keyword}%") | Item.description.ilike(f"%{keyword}%")).all()
    return render_template('browse.html', items=items)


@app.route('/edit/<int:item_id>', methods=['GET', 'POST'])
def edit_item(item_id):
    item = Item.query.get(item_id)
    if item:
        if request.method == 'POST':
            item.name = request.form['name']
            item.description = request.form['description']
            item.image = request.form['image']
            db.session.commit()
            return redirect(url_for('browse_items'))
        return render_template('edit.html', item=item)
    return render_template('edit.html', item=item)


@app.route('/remove/<int:item_id>')
def remove_item(item_id):
    item = Item.query.get(item_id)
    if item:
        db.session.delete(item)
        db.session.commit()
        return redirect(url_for('browse_items'))
    else:
        # Handle item not found
        return "Item not found"


if __name__ == '__main__':
    app.run(debug=True)
