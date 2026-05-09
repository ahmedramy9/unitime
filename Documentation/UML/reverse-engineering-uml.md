# UniTime Reverse-Engineered UML Diagrams

This bundle contains UML diagrams reverse-engineered from the local UniTime codebase. It includes both editable PlantUML sources and SVG image files that can be embedded in a report or opened directly in a browser.

## Diagrams

| Diagram | Image | Editable source | What it shows |
| --- | --- | --- | --- |
| System component UML | [system-component-diagram.svg](system-component-diagram.svg) | [system-component-diagram.puml](system-component-diagram.puml) | The web app boundary, servlet/filter chain, Spring context, major subsystems, solver integration, persistence, and external interfaces. |
| Student Scheduling subsystem class UML | [student-scheduling-subsystem-class-diagram.svg](student-scheduling-subsystem-class-diagram.svg) | [student-scheduling-subsystem-class-diagram.puml](student-scheduling-subsystem-class-diagram.puml) | One subsystem in detail: GWT service layer, online sectioning server/actions, solver services, DAOs, and persisted scheduling entities. |
| Student Scheduling request sequence UML | [student-scheduling-rpc-sequence.svg](student-scheduling-rpc-sequence.svg) | [student-scheduling-rpc-sequence.puml](student-scheduling-rpc-sequence.puml) | Runtime request flow for a course-selection/report request through filters, RPC dispatch, permissions, online sectioning or batch solver, and persistence. |

## Reverse-Engineering Basis

The diagrams were derived from these code/config areas:

- `README.md`: product scope and named functional components.
- `pom.xml`: Java web stack and core dependencies, including Struts 2, Spring Security, Hibernate, GWT, and CPSolver.
- `WebContent/WEB-INF/web.xml`: servlet mappings, filters, listeners, GWT routes, `/api/*`, `/export`, `/upload`, and report output endpoints.
- `WebContent/WEB-INF/applicationContext.xml`: Spring component scan over `org.unitime`.
- `WebContent/WEB-INF/securityContext.xml`: security integration and permission expression handling.
- `JavaSource/org/unitime/timetable/*`: top-level package boundaries. The largest observed areas are `model`, `gwt`, `server`, `onlinesectioning`, `solver`, and `action`.
- `JavaSource/org/unitime/timetable/gwt/command/server/GwtRpcServlet.java`: command-style GWT RPC dispatch to `GwtRpcImplementation` Spring beans.
- `JavaSource/org/unitime/timetable/spring/gwt/GwtDispatcherServlet.java`: GWT service dispatch by service name to Spring beans.
- `JavaSource/org/unitime/timetable/gwt/server/SectioningServlet.java`: Student Scheduling service facade, permissions, DAO use, and online sectioning action execution.
- `JavaSource/org/unitime/timetable/server/sectioning/SectioningReportsBackend.java`: report execution path through `OnlineSectioningServer` or `StudentSolverProxy`.
- `JavaSource/org/unitime/timetable/onlinesectioning/OnlineSectioningServer.java` and `OnlineSectioningAction.java`: online sectioning action model and in-memory scheduling API.
- Hibernate mappings and generated base model classes for persistent relationships: `Student.hbm.xml`, `CourseDemand.hbm.xml`, `CourseRequest.hbm.xml`, `StudentClassEnrollment.hbm.xml`, `Reservation.hbm.xml`, `InstructionalOffering.hbm.xml`, `InstrOfferingConfig.hbm.xml`, `BaseClass_.java`, and `BaseSchedulingSubpart.java`.

## Subsystem Highlight

The required subsystem is **Student Scheduling / Online Sectioning**. It is represented as:

- A runtime component in the system component diagram.
- A detailed class-level subsystem with service, solver, action, DAO, and domain-model relationships.
- A request sequence showing how an RPC/report action reaches online sectioning or the batch student solver.

