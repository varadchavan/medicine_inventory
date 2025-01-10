# medicine_inventory
 this python based web app is medicine inventory management app where users can scan medicine barcode and allot team in different sections and can custom name an container and add specific medicines to it ,it will keep record of those medicines it can also generate bill and function like medicine ERP software.
from flask import Flask, render_template, request, redirect, url_for, flash
from flask_sqlalchemy import SQLAlchemy
from forms import MedicineForm, ContainerForm
from pyzbar.pyzbar import decode
from PIL import Image

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
app.config['SECRET_KEY'] = 'your_secret_key'
db = SQLAlchemy(app)

# Import models from models.py
from models import Medicine, Container

@app.route('/')
def index():
    medicines = Medicine.query.all()
    containers = Container.query.all()
    return render_template('index.html', medicines=medicines, containers=containers)

@app.route('/add_medicine', methods=['GET', 'POST'])
def add_medicine():
    form = MedicineForm()
    if form.validate_on_submit():
        new_medicine = Medicine(
            name=form.name.data,
            barcode=form.barcode.data,
            quantity=form.quantity.data,
            price=form.price.data
        )
        db.session.add(new_medicine)
        db.session.commit()
        flash('Medicine added successfully!', 'success')
        return redirect(url_for('index'))
    return render_template('add_medicine.html', form=form)

@app.route('/add_container', methods=['GET', 'POST'])
def add_container():
    form = ContainerForm()
    if form.validate_on_submit():
        new_container = Container(
            name=form.name.data,
            medicines=form.medicines.data
        )
        db.session.add(new_container)
        db.session.commit()
        flash('Container created successfully!', 'success')
        return redirect(url_for('index'))
    return render_template('add_container.html', form=form)

@app.route('/scan_barcode', methods=['POST'])
def scan_barcode():
    file = request.files['barcode_image']
    image = Image.open(file)
    barcode_data = decode(image)
    if barcode_data:
        return barcode_data[0].data.decode('utf-8')
    else:
        flash('No barcode found!', 'error')
        return redirect(url_for('index'))

@app.route('/generate_bill', methods=['POST'])
def generate_bill():
    selected_medicines = request.form.getlist('selected_medicines')
    total_price = sum([Medicine.query.get(med_id).price for med_id in selected_medicines])
    # TODO: Generate a PDF bill and save/send it to the user
    flash(f'Bill generated: {total_price}', 'success')
    return redirect(url_for('index'))

if __name__ == '__main__':
    db.create_all()
    app.run(debug=True)
