# examApi

## Question 1 :

```
Vous développez une API REST avec ExpressJS qui sera utilisée par deux types d’utilisateurs : les
administrateurs et les utilisateurs standards. Le rôle du client est déterminé grâce au header “xadmin”,
dont la valeur est "true" pour les administrateurs, et qui est absent pour les autres
utilisateurs.
Écrivez le code permettant de restreindre l’accès aux routes sensibles de votre API en fonction du
rôle de l’utilisateur.
```

```
async function verifyAdminStatus(req, res) {
    const isAdmin = req.header('xadmin');
    if (isAdmin === undefined) {
        return res.status(400).json({ error: 'HEADER NOT FOUND' });
    }
    if (!isAdmin) {
        return res.status(403).json({ error: 'UNAUTHORIZED' });
    }
}
```
J'utiliserais cette fonction sur toutes mes routes restreintes en faisant appel de cette manière:
```
router.get('/', async (req, res) => {
    try {
        verifyAdminStatus(req, res)
        const employees = await employeeRepository.getAllEmployees();
        res.json(employees);
    } catch (error) {
        await handleErrors(error, res)
    }
});
```


## Question 2 :

Vous développez une API REST de gestion de projets avec ExpressJS et Sequelize. Cette API doit
permettre de gérer des employés et leurs projets :
• Un employé est défini par son nom, son email et sa date d’embauche.
• Un projet est défini par un titre, une description, une date de début et une date de fin
(optionnelle si le projet est en cours) et un budget.
• Un employé peut être impliqué dans plusieurs projets à la fois.
Écrivez le code Sequelize permettant de modéliser ces deux entités et leur relation.

```
const Employee = sequelizeInstance.define('employees', {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    name: {
        type: DataTypes.STRING,
        allowNull: false,
        validate: {
            notEmpty: true,
        },
    },
    email: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true,
        validate: {
            isEmail: true,
        },
    },
    entryDate: {
        type: DataTypes.DATE,
        allowNull: false,
        defaultValue: DataTypes.NOW
    },
}, {
    timestamps: true,
});


const Project = sequelizeInstance.define('projects', {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    title: {
        type: DataTypes.STRING,
        allowNull: false,
        validate: {
            notEmpty: true,
        },
    },
    description: {
        type: DataTypes.TEXT,
        allowNull: true,
    },
    startDate: {
        type: DataTypes.DATE,
        allowNull: false,
    },
    endDate: {
        type: DataTypes.DATE,
        allowNull: true,
    },
    budget: {
        type: DataTypes.FLOAT,
        allowNull: false,
        validate: {
            isFloat: true,
            min: 0,
        },
    },
}, {
    timestamps: true,
});

const EmployeeProject = sequelizeInstance.define('employee_projects', {}, {
    timestamps: false,
});

Employee.belongsToMany(Project, { through: EmployeeProject });
Project.belongsToMany(Employee, { through: EmployeeProject });
```

## Question 3 :
```
En utilisant le code de la Question 2 :
- Écrivez le code Sequelize permettant de trouver tous les projets assignés à l’employé avec l’id
  “employeeId”
- Écrivez le code Sequlize pour assigner à l’employé avec l’id “employeeId” les deux projets
  “projectId1” et “projectId2”
```

```
const employeeId = 1
const projectsIds = [1,2]

async function getProjectsByEmployee(employeeId) {
    return employeeWithProjects = await Employee.findByPk(employeeId, {
        include: Project,
    });
}

async function addProjectsToEmployee(employeeId, projectsIds){
    const employee = await Employee.findByPk(employeeId);

    if (employee) {
        await employee.addProjects(projectsIds);
    } else {
        throw new Error('EMPLOYEE_NOT_FOUND');
    }
}

const result_1 = getProjectsByEmployee(employeeId)

const result_2 = addProjectsToEmployee(employeeId, projectsIds)
```

## Question 4:

```
Décrivez en quelques phrases les étapes nécessaires pour implémenter l’authentification par Token
JWT dans une API REST (Express ou autre). Vous pouvez inclure des exemples de code pour
illustrer votre réponse, mais une explication rédigée est attendue.
```

```
Pour implémenter l’authentification par token JWT nous allons d'abord avoir besoin d'implémenter la logique qui permet 
à un utilisateur de se connecter. 
Je vais mettre en place un service qui fera la vérification des données de connexion fournies et qui crééra le JWT signé 
avec une clé secrète, dedans je pourrai fournir des infos concernants l'utilisateur et une date d'expiration.
Ensuite ce token sera vérifié à chaques requêtes que l'utilisateur fait grâce à la création d'un middleware. Le but 
de ce middleware sera de valider le token en vérifiant sa signature et sa date d'expiration. 
Pour finir je pourrai appliquer ce middleware à toutes mes routes qui nécessitent une authentification. Dans le cas où le 
token est invalide, je pourrai rediriger vers une page de connexion pour régénérer un nouveau JWT.
```