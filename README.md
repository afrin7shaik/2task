const express = require('express');
const cors = require('cors');
const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// In-memory database
let courses = [];
let nextId = 1;

// Helper function to find course by ID
const findCourse = (id) => courses.find(course => course.id === id);

// 1. CREATE - Add a new course
app.post('/api/courses', (req, res) => {
    try {
        const { title, description, duration } = req.body;

        // Validate required fields
        if (!title || !description || !duration) {
            return res.status(400).json({
                status: 'error',
                message: 'Please provide title, description, and duration'
            });
        }

        // Create new course
        const newCourse = {
            id: nextId++,
            title,
            description,
            duration,
            createdAt: new Date().toISOString()
        };

        courses.push(newCourse);
        res.status(201).json({
            status: 'success',
            data: newCourse
        });
    } catch (error) {
        res.status(500).json({
            status: 'error',
            message: 'Error creating course'
        });
    }
});

// 2. READ - Get all courses
app.get('/api/courses', (req, res) => {
    try {
        res.json({
            status: 'success',
            data: courses
        });
    } catch (error) {
        res.status(500).json({
            status: 'error',
            message: 'Error retrieving courses'
        });
    }
});

// 3. READ - Get single course by ID
app.get('/api/courses/:id', (req, res) => {
    try {
        const id = parseInt(req.params.id);
        const course = findCourse(id);

        if (!course) {
            return res.status(404).json({
                status: 'error',
                message: 'Course not found'
            });
        }

        res.json({
            status: 'success',
            data: course
        });
    } catch (error) {
        res.status(500).json({
            status: 'error',
            message: 'Error retrieving course'
        });
    }
});

// 4. UPDATE - Update a course
app.put('/api/courses/:id', (req, res) => {
    try {
        const id = parseInt(req.params.id);
        const courseIndex = courses.findIndex(course => course.id === id);

        if (courseIndex === -1) {
            return res.status(404).json({
                status: 'error',
                message: 'Course not found'
            });
        }

        const { title, description, duration } = req.body;
        
        // Update course while preserving existing data
        courses[courseIndex] = {
            ...courses[courseIndex],
            title: title || courses[courseIndex].title,
            description: description || courses[courseIndex].description,
            duration: duration || courses[courseIndex].duration,
            updatedAt: new Date().toISOString()
        };

        res.json({
            status: 'success',
            data: courses[courseIndex]
        });
    } catch (error) {
        res.status(500).json({
            status: 'error',
            message: 'Error updating course'
        });
    }
});

// 5. DELETE - Delete a course
app.delete('/api/courses/:id', (req, res) => {
    try {
        const id = parseInt(req.params.id);
        const courseIndex = courses.findIndex(course => course.id === id);

        if (courseIndex === -1) {
            return res.status(404).json({
                status: 'error',
                message: 'Course not found'
            });
        }

        courses.splice(courseIndex, 1);
        res.status(204).send();
    } catch (error) {
        res.status(500).json({
            status: 'error',
            message: 'Error deleting course'
        });
    }
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
