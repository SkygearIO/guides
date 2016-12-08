---
title: Restful HTTP Endpoint
---

While you can easily perform the CRUD (Create, Read, Update, Delete)
operations by using the SDK directly, you still have the option
to implement a custom [restful][restful-wiki] backend using cloud codes.
The decorator `skygear.rest` helps you create the necessary routes
for this purpose.

### Using the decorator

To implement the restful resource, you need to create a `class`
that inherits one of the Skygear restful classes:

- **`skygear.RestfulResource`**
  
  Using the `RestfulResource` class allows you to define the methods
  shown in the example below, which correspond to the restful endpoints
  noted in the comments:

  ```python
  import skygear

  @skygear.rest('/notes')
  class MyNote(skygear.RestfulResource):

      # GET /notes
      def index(self):
          # return list of notes
          pass

      # POST /notes
      def create(self):
          payload = self.get_payload()
          # process the payload and save the notes to database

      # DELETE /notes/{id}
      def delete(self, id):
          # delete the item of the given id
          pass

      # PUT /notes/{id}
      def update(self, id):
          payload = self.get_payload()
          # process the payload and update the notes in the database

      # GET /notes/{id}
      def get(self, id):
          pass
  ```

  In the above example, the mapping between the class methods and
  the HTTP endpoints are:

  | class method | HTTP request method | endpoint | description |
  |---|---|---|---|
  | `index(self)` | GET | /notes | list notes |
  | `create(self)` | POST | /notes | create a note |
  | `delete(self, id)` | DELETE | /notes/{id} | delete the note of given id |
  | `update(self, id)` | PUT | /notes/{id} | update the note of given id |
  | `get(self, id)` | GET | /notes/{id} | fetch the note of given id |

  <div class="caution">

  **Caution:**

  By default, these endpoints are accessible by anyone,
  including unauthenticated user.
  To restrict access to authenticated user, specify the `user_required`
  to be `True` in the decorator. And you can obtain the user ID by
  `skygear.util.context.current_user_id()`.

  ```python
  @skygear.rest('/notes', user_required=True)
  ```

  </div>

- **`skygear.RestfulRecord`**

  Using the `RestfulRecord` class will have the 5 restful API methods
  shown above implemented for you. By subclassing `RestfulRecord`,
  you need to set the class variables `record_type` and `database_id` to specify
  the associated record type and database ID.
  
  ```python`
  @skygear.rest('/tasks')
  class RestfulTask(skygear.RestfulRecord):
      record_type = 'task'
      database_id = '_public'
  ```

  On top of the implemented methods, you can implement
  the custom methods `query_options` and `predicate` to tweak
  the behavior of the listing method `index()`.

  - `query_options` - This method should return a dict containing parameters
    to be sent to the Skygear Server `record:query action`.
  - `predicate` - This method can be used to filter the resources from
    Skygear Server. It should return an array of Skygear query predicates.

  The example below shows the implementation of a restful API with
  pagination support.

  ```python
  import skygear

  @skygear.rest('/tasks')
  class MyTask(skygear.RestfulRecord):
      record_type = 'task'
      database_id = '_public'

      def query_options(self):
          # /tasks?limit=50&offset=100 will return the 101th to 150th records
          # ordered by the `due_date` field
          return {
              'limit': int(self.request.values.get('limit', 100)),
              'offset': int(self.request.values.get('offset', 0)),
              'sort': [[{'$val': 'due_date', '$type': 'keypath'}, 'asc']],
              'count': True,
          }

      def predicate(self):
          # Only tasks belonged to the "work" `category` will be returned
          return ['eq', {'$type': 'keypath', '$val': 'category'}, 'work']
  ```

  <div class="note">

  **Note:**

  The endpoints created with `RestfulRecord` are subject to the usual
  access control, i.e.:

  - Only authenticated users can create, update or delete records
  - Access control defined for the records are applied

  ```python
  @skygear.rest('/notes', user_required=True)
  ```

  </div>

[restful-wiki]: https://en.wikipedia.org/wiki/Representational_state_transfer#Applied_to_web_services
