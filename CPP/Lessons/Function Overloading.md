Function overloading is a powerful way to enable polymorphism for a class method.

A good example is shown here for a class called Joint, which represents a single Denavit Hartenberg Joint.

We may have a constructor method. I have defined a method to be initialised by input, and a second default constructor that overloads the method if no inputs are provided:

.h file:

```
	Joint(float theta, float alpha, float a, float d); // Constructor
	Joint(); // Default constrcutor
```

.cpp file:
```
// Constructor
Joint::Joint(float theta, float alpha, float a, float d)
	: _theta(deg2rad(theta)), _alpha(deg2rad(alpha)), _a(a), _d(d), _Frame(Eigen::Matrix4f::Zero()) {						// Member initialisation list - inits the class members. Note angles are immediately converted to rad
	
	Joint::evaluateFrame();
}

// Defualt constructor																									
Joint::Joint()																												// COnstructor overloading - this object is instantiated when no iput arguements are provided
	: _theta(0.0f), _alpha(0.0f), _a(0.0f), _d(0.0f), _Frame(Eigen::Matrix4f::Identity()) {
	evaluateFrame();
}
```