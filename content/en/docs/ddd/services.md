---
title: "Services"
linkTitle: "Services"
weight: 4
description: >
  How to use services
---

## Services

After you complete the domain models, you can start building the services. The following describe a service:
- It performs a significant piece of business logic within the domain.
- It calculates some values.
- It interacts with the repository layer.

Try to push down as much logic as possible into your domain objects (i.e. create service functions within the domain `.go` files.)

Domain services provide services within the domain, while application services are used to compose other services and repositories--they do not contain domain logic.

### Example

A student wants to register for a class. You have the `Student` entity:
```go
type Student struct {
	ID        int
	firstName string
	lastName  string
	courses   []Class
}
```
And you have the `Class` entity:
```go
type Class struct {
	ID            int
	Name          string
	students      []Student
	maxEnrollment int
}
```
Next, we want to register a student for the class. You might consider creating a `Student.AddClass` or `Class.AddStudent` method, but either of those methods would require that you reference another entity and perform business logic. For example:
```go
// This is bad
func (s *Student) AddClass(c *Class) error {
    // validation logic
    if len(c.students) > c.maxEnrollment {
        return errors.New("class is full")
    }
    // ...
    // Class business logic
    c.append(c.students, s)
    return nil
}
```
The solution is to add a `RegistrationService`. First, define the struct:
```go
// RegistrationService registers students for the class.
type RegistrationService struct {
	class *Class
}
```
Next, create a constructor that injects a Class instance into the service:
```go
func NewRegistrationService(c *Class) *RegistrationService {
	return &RegistrationService{class: c}
}
```
Finally, define the `RegisterStudent` method on the `RegistrationService`:
```go
func (r *RegistrationService) RegisterStudent(s *Student) error {
	if len(r.class.students) > r.class.maxEnrollment {
		return errors.New("class is full")
	}
	if r.class.contains(s) {
		return errors.New("student is already enrolled in this class")
	}
	r.class.students = append(r.class.students, *s)
	return nil
}
```