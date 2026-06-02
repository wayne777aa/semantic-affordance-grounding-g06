# Homework 5: Ontology-based Semantic Grounding

## Project Title

Group 06 Semantic Grounding Ontology for AI Capstone Tabletop Manipulation Tasks

## Group Members

* Member 1: 劉逢穎 112550077
* Member 2: 蔡浚庭 112550005
* Member 3: 徐振豪 112550063
* Member 4: 蔡懷恩 112550020
* Member 5: 何柏翰 112550028

## Selected Task

Our group focuses on the **Cutlery Arrangement** task from the AI Capstone course project. In this task, a robot arranges a knife and a fork around a plate.

Although our main task is cutlery arrangement, this repository also includes the required baseline object instances for the other predefined course tasks:

1. **Cup stacking**

   * blue cup
   * pink cup

2. **Cutlery arrangement**

   * knife
   * fork
   * plate

3. **Toy block collection**

   * toy blocks
   * basket

The main semantic question answered by this repository is:

Which objects in the environment are graspable, and why?

## Repository Structure

```text
semantic-affordance-grounding/
├── README.md
├── report.md
├── ontology/
│   ├── group-ontology.ttl
│   ├── inferred-results.ttl
│   └── imports/
│       ├── course-affordance.ttl
│       └── course-alignment.ttl
├── queries/
│   ├── graspable_objects.rq
│   └── task_objects.rq
├── results/
│   ├── graspable_objects_output.txt
│   ├── task_objects_output.txt
│   └── screenshots/
│       ├── protege_reasoning_result.png
│       └── protege_sparql_result.png
├── src/
│   └── ...
└── LICENSE
```

## Authored and Imported Resources

The group-authored ontology is:

```text
ontology/group-ontology.ttl
```

The imported course ontology resources are:

```text
ontology/imports/course-affordance.ttl
ontology/imports/course-alignment.ttl
```

The course ontology provides the shared AI Capstone vocabulary, including object classes, task-role classes, affordance classes, and semantic properties. Our group ontology reuses these terms instead of redefining them locally.

The inferred graph after reasoning is saved as:

```text
ontology/inferred-results.ttl
```

## Namespace Policy

The shared course namespace is:

```turtle
@prefix cap: <https://hcis.io/ontology/aicapstone/2026/> .
```

The Group 06 namespace is:

```turtle
@prefix g06: <https://hcis.io/ontology/aicapstone/2026/group06/> .
```

Course-level terms such as `cap:Cup`, `cap:Knife`, `cap:Fork`, `cap:Plate`, `cap:ToyBlock`, `cap:Basket`, `cap:PhysicalObject`, `cap:GraspingAffordance`, and `cap:GraspableObject` are reused from or aligned with the course ontology.

Group-specific individuals such as `g06:knife01`, `g06:fork01`, `g06:plate01`, `g06:blueCup01`, `g06:pinkCup01`, `g06:block01`, and `g06:basket01` are defined under the `g06:` namespace.

## Ontology Design

The ontology distinguishes five modeling layers:

| Layer          | Meaning                                     | Example                                                                                                      |
| -------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Object type    | Shared class for a kind of object           | `cap:Knife`, `cap:Fork`, `cap:Cup`                                                                           |
| Task role      | Role played by an object in a task          | `cap:TargetObject`, `cap:ReferenceObject`, `cap:CollectableObject`, `cap:ContainerTarget`                    |
| Affordance     | Action-relevant object capability           | `cap:GraspingAffordance`, `cap:SupportAffordance`, `cap:ContainmentAffordance`, `cap:StackabilityAffordance` |
| Instance       | Group-specific observed or simulated object | `g06:knife01`, `g06:fork01`, `g06:blueCup01`                                                                 |
| Inferred class | Class membership derived by reasoning       | `cap:GraspableObject`                                                                                        |

The ontology does not treat every task-relevant object as graspable. For example, the plate is modeled as a reference/support object in the cutlery arrangement task, and the basket is modeled as a container target in the toy block collection task. They are task-relevant, but they are not direct grasp targets in our modeling choice.

## Modeled Objects and Task Roles

| Object instance | Object type    | Task role               | Main semantic interpretation                      |
| --------------- | -------------- | ----------------------- | ------------------------------------------------- |
| `g06:knife01`   | `cap:Knife`    | `cap:TargetObject`      | Direct manipulation target in cutlery arrangement |
| `g06:fork01`    | `cap:Fork`     | `cap:TargetObject`      | Direct manipulation target in cutlery arrangement |
| `g06:plate01`   | `cap:Plate`    | `cap:ReferenceObject`   | Placement/reference object for cutlery            |
| `g06:blueCup01` | `cap:Cup`      | `cap:TargetObject`      | Required cup-stacking object                      |
| `g06:pinkCup01` | `cap:Cup`      | `cap:ReferenceObject`   | Required cup-stacking reference object            |
| `g06:block01`   | `cap:ToyBlock` | `cap:CollectableObject` | Collectable toy block                             |
| `g06:block02`   | `cap:ToyBlock` | `cap:CollectableObject` | Collectable toy block                             |
| `g06:block03`   | `cap:ToyBlock` | `cap:CollectableObject` | Collectable toy block                             |
| `g06:basket01`  | `cap:Basket`   | `cap:ContainerTarget`   | Container target for toy blocks                   |

## Key Reasoning Pattern

The key inferable class is `cap:GraspableObject`.

In Description Logic style, the intended definition is:

```text
cap:GraspableObject ≡ cap:PhysicalObject
                     ⊓ ∃ cap:hasAffordance.cap:GraspingAffordance
```

This means that an object can be classified as a `cap:GraspableObject` if it is a physical object and has at least one grasping affordance.

In Turtle, this is represented with an `owl:equivalentClass` pattern using `owl:intersectionOf` and `owl:someValuesFrom`.

The important point is that the group ontology does not manually assert final graspability results such as:

```turtle
g06:fork01 a cap:GraspableObject .
```

Instead, the ontology asserts object types, task roles, labels, pose frames, and affordance-related facts. The reasoner derives `cap:GraspableObject` membership from the OWL class definition.

## What Is Asserted and What Is Inferred

Asserted facts include examples such as:

```turtle
g06:fork01 a cap:Fork .
g06:fork01 cap:hasTaskRole cap:TargetObject .
g06:fork01 cap:hasObjectLabel "fork" .
g06:fork01 cap:hasPoseFrame "world/object_fork" .
```

The inferred result is:

```turtle
g06:fork01 a cap:GraspableObject .
```

This inference is supported because `cap:Fork` is a physical object class associated with grasping affordance conditions, and `cap:GraspableObject` is defined as a physical object with some grasping affordance.

## Reasoning Workflow

The reasoning workflow was checked using Protégé with an OWL reasoner. The reasoner classified object classes such as `cap:Cup`, `cap:Fork`, `cap:Knife`, and `cap:ToyBlock` under `cap:GraspableObject`.

The inferred graph was exported and saved as:

```text
ontology/inferred-results.ttl
```

The inferred graph is then queried using SPARQL.

Screenshots of the Protégé reasoning result and SPARQL result are included under:

```text
results/screenshots/
```

## Query Execution

The required query is:

```text
queries/graspable_objects.rq
```

This query retrieves inferred `cap:GraspableObject` individuals:

```sparql
PREFIX cap: <https://hcis.io/ontology/aicapstone/2026/>

SELECT DISTINCT ?obj ?label ?role
WHERE {
  ?obj a cap:GraspableObject .
  OPTIONAL { ?obj cap:hasObjectLabel ?label . }
  OPTIONAL { ?obj cap:hasTaskRole ?role . }
}
ORDER BY ?obj
```

The query was executed over the inferred graph, not only over the raw asserted ontology.
The results can be found under /results/screenshots/QueryResult.png

## Expected Query Output

The expected inferred graspable objects are:

```text
g06:blueCup01
g06:pinkCup01
g06:fork01
g06:knife01
g06:block01
g06:block02
g06:block03
```

The expected non-graspable task-relevant objects in our modeling are:

```text
g06:plate01
g06:basket01
```

The plate and basket are not returned by the graspable-object query because they are modeled as a reference/support object and a container target, respectively, rather than direct grasp targets.

The saved query output is:

```text
results/graspable_objects_output.txt
```

## Limitations

This ontology models semantic graspability at the task-concept level. It does not model full physical feasibility. For example, it does not check object mass, exact 3D geometry, collision constraints, grasp pose stability, gripper force limits, or policy success probability.

Therefore, `cap:GraspableObject` means that the object is semantically grounded as graspable in the course ontology, not that every physical grasp attempt will succeed in all robot states.
