## Usefool links

- [SQLAlchemy — Python Tutorial](https://towardsdatascience.com/sqlalchemy-python-tutorial-79a577141a91)
- [Constructing Database Queries with SQLAlchemy](https://hackingandslacking.com/constructing-database-queries-with-sqlalchemy-9907f4f8e04b)
- [Python Database Management with SQLAlchemy](https://hackingandslacking.com/pythonic-database-management-with-sqlalchemy-3a6b99b102a0)
- [SQLAlchemy pythonsheets](https://www.pythonsheets.com/notes/python-sqlalchemy.html)
- [Индексы PostgreSQL: полное руководство](https://ru.haru-atari.com/blog/6-indexes-in-postgresql-full-manual)
- [Using Python SQLAlchemy session in multithreading](https://copdips.com/2019/05/using-python-sqlalchemy-session-in-multithreading.html)

The last part of our query determines how many rows to return, and the nature of how those rows are determined:
- `all()` will return all records which match our query.
- `first()` returns the first record in order of appearance.
- `one()` returns a single value (not necessarily the first).
- `scalar()` returns a single value if one exists, None if no values exist, or raises an exception if multiple records are returned.
- `get([VALUE(S)])` searches against a model's primary key to return rows where the primary key is equal to the value provided. get() also accepts tuples in the event that multiple foreign keys should be searched. Lastly, get() can also accept a dictionary and will return rows where the columns (dictionary keys) match the values provided.

**where**

	SQL :
	SELECT * FROM census 
	WHERE sex = F
	SQLAlchemy :
	db.select([census]).where(census.columns.sex == 'F')

**in**

	SQL :
	SELECT state, sex
	FROM census
	WHERE state IN (Texas, New York)
	SQLAlchemy :
	db.select([census.columns.state, census.columns.sex]).where(census.columns.state.in_(['Texas', 'New York']))

**and, or, not**

	SQL :
	SELECT * FROM census
	WHERE state = 'California' AND NOT sex = 'M'
	SQLAlchemy :
	db.select([census]).where(db.and_(census.columns.state == 'California', census.columns.sex != 'M'))

**order by**

	SQL :
	SELECT * FROM census
	ORDER BY State DESC, pop2000
	SQLAlchemy :
	db.select([census]).order_by(db.desc(census.columns.state), census.columns.pop2000)

**functions**

	SQL :
	SELECT SUM(pop2008)
	FROM census
	SQLAlchemy :
	db.select([db.func.sum(census.columns.pop2008)])

**group by**

	SQL :
	SELECT SUM(pop2008) as pop2008, sex
	FROM census
	SQLAlchemy :
	db.select([db.func.sum(census.columns.pop2008).label('pop2008'), census.columns.sex]).group_by(census.columns.sex)

**distinct**

	SQL :
	SELECT DISTINCT state
	FROM census
	SQLAlchemy :
	db.select([census.columns.state.distinct()])

**case & cast**

	female_pop = db.func.sum(db.case([(census.columns.sex == 'F', census.columns.pop2000)],else_=0))

**joins**

If you have two tables that already have an established relationship, you can automatically use that relationship by just adding the columns we want from each table to the select statement.

	select([census.columns.pop2008, state_fact.columns.abbreviation])


	query = db.select([census, state_fact])
	query = query.select_from(census.join(state_fact, census.columns.state == state_fact.columns.name))
	results = connection.execute(query).fetchall()
	df = pd.DataFrame(results)
	df.columns = results[0].keys()
	df.head(5)


**Joins with aliases**

	class User(Model):
	    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
	    username = Column(db.String(80), unique=True, nullable=False)
	    skills = db.relationship('UserSkill')

	class Skill(Model):
	    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
	    name = Column(db.String(80))

	class UserSkill(Model):
	    status = db.Column(db.Enum(SkillStatus))
	    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), primary_key=True)
	    skill_id = db.Column(db.Integer, db.ForeignKey('skills.id'), primary_key=True)
	    skill = db.relationship("Skill")

	userSkillF = aliased(UserSkill)
	userSkillI = aliased(UserSkill)
	skillF = aliased(Skill)
	skillI = aliased(Skill)

	db.session.query(User.id, User.username,\
	         func.group_concat(func.distinct(skillF.name)).label('skills'),\
	         func.group_concat(func.distinct(skillI.name)).label('other_skills')).\
	    join(userSkillF, User.skills).\
	    join(userSkillI, User.skills).\
	    join(skillF, userSkillF.skill).filter(skillF.id.in_(skillIds)).\
	    join(skillI, userSkillI.skill).\
	    group_by(User.id).all()


**insert**

	from sqlalchemy import create_engine
	from sqlalchemy import MetaData
	from sqlalchemy import Table
	from sqlalchemy import Column
	from sqlalchemy import Integer
	from sqlalchemy import String

	db_uri = 'sqlite:///db.sqlite'
	engine = create_engine(db_uri)

	# create table
	meta = MetaData(engine)
	table = Table('user', meta,
	   Column('id', Integer, primary_key=True),
	   Column('l_name', String),
	   Column('f_name', String))
	meta.create_all()

	# insert data via insert() construct
	ins = table.insert().values(
	      l_name='Hello',
	      f_name='World')
	conn = engine.connect()
	conn.execute(ins)

	# insert multiple data
	conn.execute(table.insert(),[
	   {'l_name':'Hi','f_name':'bob'},
	   {'l_name':'yo','f_name':'alice'}])

