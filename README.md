# Data Engineer Coding Challenge - Family Tree

## Candidate

Remo Fernandes S

## Approach

* Designed using OOP principles
* Separate classes:

  * Person (data)
  * FamilyTree (structure)
  * RelationshipService (logic)

## Features

* Add child through mother
* Retrieve:

  * Siblings
  * Brothers / Sisters
  * Cousins
  * Maternal Uncles

## How to Run

```bash
python src/main.py input.txt
```

## Sample Output

```
CHILD_ADDITION_SUCCEEDED
Bill Charlie Percy
Bill Charlie Percy
NONE
NONE
```

## Design Decisions

* Used dictionary for fast lookup
* Modular structure for scalability
* Easy to extend for new relationships



class Person:
    def __init__(self, name, gender):
        self.name = name
        self.gender = gender
        self.spouse = None
        self.children = []

from person import Person

class FamilyTree:
    def __init__(self):
        self.members = {}

    def add_person(self, name, gender):
        person = Person(name, gender)
        self.members[name] = person
        return person

    def add_spouse(self, name1, name2):
        p1 = self.members.get(name1)
        p2 = self.members.get(name2)
        if p1 and p2:
            p1.spouse = p2
            p2.spouse = p1

    def add_child(self, mother_name, child_name, gender):
        if mother_name not in self.members:
            return "PERSON_NOT_FOUND"

        mother = self.members[mother_name]

        if mother.gender != "Female":
            return "CHILD_ADDITION_FAILED"

        child = Person(child_name, gender)
        self.members[child_name] = child

        mother.children.append(child)

        if mother.spouse:
            mother.spouse.children.append(child)

        return "CHILD_ADDITION_SUCCEEDED"

    def get_mother(self, name):
        for person in self.members.values():
            for child in person.children:
                if child.name == name:
                    return person
        return None


  class RelationshipService:
    def __init__(self, tree):
        self.tree = tree

    def get_siblings(self, name):
        if name not in self.tree.members:
            return ["PERSON_NOT_FOUND"]

        mother = self.tree.get_mother(name)
        if not mother:
            return []

        return [child.name for child in mother.children if child.name != name]

    def get_brothers(self, name):
        siblings = self.get_siblings(name)
        if siblings == ["PERSON_NOT_FOUND"]:
            return siblings

        return [p for p in siblings if self.tree.members[p].gender == "Male"]

    def get_sisters(self, name):
        siblings = self.get_siblings(name)
        if siblings == ["PERSON_NOT_FOUND"]:
            return siblings

        return [p for p in siblings if self.tree.members[p].gender == "Female"]

    def get_cousins(self, name):
        if name not in self.tree.members:
            return ["PERSON_NOT_FOUND"]

        mother = self.tree.get_mother(name)
        if not mother:
            return []

        grand_mother = self.tree.get_mother(mother.name)
        if not grand_mother:
            return []

        cousins = []
        for aunt_uncle in grand_mother.children:
            if aunt_uncle.name != mother.name:
                cousins.extend([child.name for child in aunt_uncle.children])

        return cousins

    def get_maternal_uncles(self, name):
        if name not in self.tree.members:
            return ["PERSON_NOT_FOUND"]

        mother = self.tree.get_mother(name)
        if not mother:
            return []

        grand_mother = self.tree.get_mother(mother.name)
        if not grand_mother:
            return []

        return [
            p.name for p in grand_mother.children
            if p.gender == "Male" and p.name != mother.name
        ]

import sys
from family_tree import FamilyTree
from relationship_service import RelationshipService

def process_command(tree, service, command):
    parts = command.strip().split()

    if parts[0] == "ADD_CHILD":
        result = tree.add_child(parts[1], parts[2], parts[3])
        print(result)

    elif parts[0] == "GET_RELATIONSHIP":
        name = parts[1]
        relation = parts[2]

        if relation == "Siblings":
            result = service.get_siblings(name)
        elif relation == "Brothers":
            result = service.get_brothers(name)
        elif relation == "Sisters":
            result = service.get_sisters(name)
        elif relation == "Cousins":
            result = service.get_cousins(name)
        elif relation == "Maternal-Uncle":
            result = service.get_maternal_uncles(name)
        else:
            result = []

        print(" ".join(result) if result else "NONE")

def initialize_tree(tree):
    tree.add_person("Arthur", "Male")
    tree.add_person("Margaret", "Female")
    tree.add_spouse("Arthur", "Margaret")

    tree.add_child("Margaret", "Bill", "Male")
    tree.add_child("Margaret", "Charlie", "Male")
    tree.add_child("Margaret", "Percy", "Male")

def main():
    if len(sys.argv) < 2:
        print("Usage: python src/main.py input.txt")
        return

    file_path = sys.argv[1]

    tree = FamilyTree()
    initialize_tree(tree)

    service = RelationshipService(tree)

    with open(file_path) as f:
        for line in f:
            process_command(tree, service, line)

if __name__ == "__main__":
    main()

ADD_CHILD Margaret Diana Female
GET_RELATIONSHIP Diana Siblings
GET_RELATIONSHIP Diana Brothers
GET_RELATIONSHIP Diana Cousins
GET_RELATIONSHIP Diana Maternal-Uncle

# No external dependencies











* CHILD_ADDITION_FAILED
* NONE
