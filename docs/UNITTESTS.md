# Adding the unit tests

For adding in the unit tests we'll make use of the Python [unittest](https://docs.python.org/3/library/unittest.html) library.

### Step 1 - Create test files

We'll create all our tests in a **tests** directory

```
mkdir tests
```

Create a new directory called **tests** and within there place two files called `__init__.py` and `test_getbooks.py`

The `__init__.py` file can be left blank. 

### Step 2 - Write a few unit tests

Create a few Python unit tests. Within the `test_getbooks.py` file, pop in the following contents:

```
import unittest

from api import app

class GetBooksTest(unittest.TestCase):

    def setUp(self):
        self.app = app.test_client()

    def test_successful_getbooks(self):

        response = self.app.get('/books', headers={"Content-Type": "application/json"})
        self.assertEqual(200, response.status_code)

    def test_successful_getbookssize(self):

        response = self.app.get('/books', headers={"Content-Type": "application/json"})
        self.assertEqual(3, len(response.json['books']))
```

**Note** You might need to tweak the test if your API returns more than 3 books 🙌

You can see a full version of this setup in [this repository here](../tests)

Thats the unit tests ready to go, now head back to the README and move on to [Step 2 - CircleCI setup](../README.md)