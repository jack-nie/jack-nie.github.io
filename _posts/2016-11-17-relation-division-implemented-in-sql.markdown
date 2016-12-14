---
layout: post
title:  "Relation devisions impelemented in sql"
date:   "2016-11-17"
keywords: ["postgresql"]
description: "postgresql"
category: "postgresql"
tags: ["postgresql"]
---
{% include JB/setup %}
除法是写为R ÷ S的二元关系。其结果由R中元组到唯一于R的属性名字（就是说只在R表头中而不在S表头中的属性）的限制构成，并且它们与S中的元组的所有组合都存在于R中。例如下面的“完成”和“DB项目”和它们的除法：

完成

| Student | Task      |
|:--------|:----------|
| Fred    | Database1 |
| Fred    | Database2 |
| Fred    | Compiler1 |
| Eugene  | Database1 |
| Eugene  | Compiler1 |
| Sara    | Database1 |
| Sara    | Database2 |

DB项目

| Task      |
|:----------|
| Database1 |
| Database2 |

完成÷ DB项目

| Student |
|:--------|
| Fred    |
| Sara    |

如果“DB项目”包含数据库项目的所有任务，则这个除法的结果精确的包含已经完成了数据库项目的所有学生。


### 数据准备

首先创建两张表，一张记录飞行员名称和飞机名称的表，这个表代表R。另一张表记录飞行员的名称，代表S。


```
CREATE TABLE hangar (
  plane_name character(15) NOT NULL
);

ALTER TABLE hangar OWNER TO postgres;

--
-- Data for Name: hangar; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY hangar (plane_name) FROM stdin;
B-1 Bomber
B-52 Bomber
F-14 Fighter
\.

--
-- Name: hangar_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY hangar
  ADD CONSTRAINT hangar_pkey PRIMARY KEY (plane_name);

--
-- PostgreSQL database dump complete

CREATE TABLE pilotskills (
  pilot_name character(15) NOT NULL,
  plane_name character(15) NOT NULL
);

ALTER TABLE pilotskills OWNER TO postgres;

--
-- Data for Name: pilotskills; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY pilotskills (pilot_name, plane_name) FROM stdin;
Celko Piper Cub
Higgins B-52 Bomber
Higgins F-14 Fighter
Higgins Piper Cub
Jones B-52 Bomber
Jones F-14 Fighter
Smith B-1 Bomber
Smith B-52 Bomber
Smith F-14 Fighter
Wilson B-1 Bomber
Wilson B-52 Bomber
Wilson F-14 Fighter
Wilson F-17 Fighter
\.

--
-- Name: pilotskills_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY pilotskills
  ADD CONSTRAINT pilotskills_pkey PRIMARY KEY (pilot_name, plane_name);
```

### Division with a Remainder

```
SELECT DISTINCT pilot_name
  FROM PilotSkills AS PS1
  WHERE NOT EXISTS
      (SELECT *
          FROM Hangar
        WHERE NOT EXISTS
              (SELECT *
                  FROM PilotSkills AS PS2
                WHERE (PS1.pilot_name = PS2.pilot_name)
                  AND (PS2.plane_name = Hangar.plane_name)));
```

```
SELECT PS1.pilot_name
  FROM PilotSkills AS PS1, Hangar AS H1
  WHERE PS1.plane_name = H1.plane_name
  GROUP BY PS1.pilot_name
  HAVING COUNT(PS1.plane_name) = (SELECT COUNT(plane_name) FROM Hangar);
```

### 参考文献
