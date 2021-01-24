# Updating and Deleting Data

This video will finish full CRUD capability.

## Basics

To update a resource you can use PUT or POST. The thing here is that you need some way to identify what resource you want to update and how you want to update it.

The easiest, in my oppinion, is to identify a resource by an ID and replace the entire row with new data (keeping the same ID).

Because replacing a resource numerous times will give us the same result, this process is considered idempotent and is suitable for PUT requirements.

## Update a Resource

When we update a resource, we just change the attributes on the object. We can get the new values from the request body.

There is no universally agreed on thing that should be returned. Here we will just return the updated object.

```python3
@app.route('/api/testimonials/<id>', methods=['POST', 'PUT'])
def update_testimonial(id):
    testimonial = Testimonial.query.get(id)

    if not testimonial:
        return {'error': 'testimonial not found'}, 400

    data = request.get_json()
    testimonial.name = data.get('name')
    testimonial.testimonial = data.get('testimonial')
    result = db.session.commit()
    return jsonify(testimonial)
```

## Delete a Resource

Not much to say, here!

```python3
@app.route('/api/testimonials/<id>', methods=['DELETE'])
def delete_testimonial(id):
    testimonial = Testimonial.query.get(id)

    if not testimonial:
        return {'error': 'testimonial not found'}, 400

    db.session.delete(testimonial)
    db.session.commit()

    return {}
```

You will then want to commit these changes and try it on the production server.
