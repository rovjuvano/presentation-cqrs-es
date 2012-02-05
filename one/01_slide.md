!SLIDE
# From CRUD to CQRS+ES

!SLIDE
### Agenda
* CRUD
* CQRS
* Event Sourcing
* Review
* More Info
* Q&A

!SLIDE
# CRUD

!SLIDE
# Blogs made easy!
# Ready in 15 minutes!

!SLIDE
## layered architecture
* Database
* Model (via ORM)
* Controller
* View/Client

!SLIDE
## Problems with CRUD

!SLIDE
* tight coupling Model <=> DB
* two data models!
* AR is a chimera

!SLIDE
* In a world with only four verbs, nouns dominate!
* Rails has 7 on resources (CQS)
* Commands: create, update, destory
* Queries: index, new, show, edit

!SLIDE
* difficult to query 3NF DB
* error prone
* joins are slow (can create temp tables)
* weakness in ORMs
* many round trips (post, comments, blogroll, etc.)
* hard to cache

!SLIDE
* write vs read optimizations
* data is discarded before writing (e.g. intent)
* data is erased after writing (e.g. the old value)
* cannot query missing data
* audit logs if not used are not reliable

!SLIDE
* bugs screw up the one and only authoritative source of data
* backups don't help much
* you are more likely to corrupt your data with code than lose all of the data
* one you will notice quickly, the other may take much time
* can be very expensive (and error prone) to cleanup

!SLIDE
* business logic usually implemented in wetware
* long UI to raw table data
* models validate and authorize, but little else
* business process are still manual - FAIL!
* CRUD > PHPMyAdmin (slightly)

!SLIDE
* as DB grows, schema changes become more risky and more impacting

!SLIDE
* Concurrency

!SLIDE
# CRUD
# CRUD
# CRUD
# CRUD

!SLIDE
# Oh No!
# Too much CRUD!
# All at once!

!SLIDE
# Write Contention
# Kills Performance
# Opposes Scalability
# Breaks Abstractions

!SLIDE
# enough already!

!SLIDE
# CQRS

!SLIDE
# CQS
* Bertrand Meyer (1988)
* Command-Query Separation

!SLIDE
## CQS (cont.)
* command performs an action
* query returns data to the caller
* not both

!SLIDE
## CQS (cont.)
* asking a question should not change the answer
* methods should return a value only if they are referentially transparent and hence possess no side effects.

!SLIDE
## CQS by other names
* commands, procedures, modifiers, mutators, statements
* queries, (pure) functions, expressions

!SLIDE
# CQS applied to CRUD

!SLIDE
# CUD

!SLIDE
# R
# R
# R
# R

!SLIDE
# CQRS
# Greg Young
# Command Query Responsibility Segregation

!SLIDE
## CQRS = CQS for Objects
## (that's it - it is not an architecture)

!SLIDE
* All methods are commands
### OR
* All methods are queries
### not both

!SLIDE
# Why?

!SLIDE
# layered architecture revisited

!SLIDE
# Read side
* DB
* ViewModel (via DBI)
* View/Client

!SLIDE
# fewer layers
# simpler layers

!SLIDE
# Write side
* DB
* Model (via ORM)
* Controller
* View/Client

!SLIDE
# no structural changes

!SLIDE
# BUT

!SLIDE
## write side is free of reads

!SLIDE
# Apply CQS to DB

!SLIDE
## write only* DB
### OR
## read only* DB
### not both
\* (from application)

!SLIDE
## write side can have optimized 3NF DB

!SLIDE
    @@@ ruby
    has_many :articles
    has_many :comments, :through => :articles

!SLIDE
## read side can be denormalized
    @@@ sql
    SELECT * FROM dashboard;

!SLIDE
## or be document store
    @@@ ruby
    ### CoachDB example

!SLIDE
# Many Specialized Read DBs
* Object Store
* Memory Image
* ViewModel
* Read DB = Cache

!SLIDE
# BUT

!SLIDE
## Now there are 2+ DBs?

!SLIDE
# &lt;aside>

!SLIDE
## DRY is sacred!

!SLIDE
# Duplicate code
# vs
# Duplicate data

!SLIDE
# DBTL
## Don't
## Be
## Too
## Literal

!SLIDE
# Backups!
# Replicas!
# Load Balancing!
# High Availability!

!SLIDE
## Duplication can be good!

!SLIDE
# &lt;/aside>

!SLIDE
# Synching Query DBs to Command DB

!SLIDE
# Event Sourcing

!SLIDE
# CUD -> Event

!SLIDE
# Update Query DBs from Events

!SLIDE
# picture

!SLIDE
# BONUS!

!SLIDE
# Store Events

!SLIDE
# Replay!
* rebuild query DBs
* create new query DBs
* rewind pre-bug and replay events

!SLIDE
# Uh-Oh!
## Now we have two authoritative data sources
### (command DB and event store)

!SLIDE
# DROP DB (and ORM)

!SLIDE
# Build Domain Models from Events
    @@@ ruby
    def on_some_event(event)
      @state = function(@state, event)
    end
### no more getter/setters
### only events change state
### encapsulation acheived!

!SLIDE
# Event Store > Audit Log
# Authoritative Data Source

!SLIDE
# Event Stores are Easy and Simple!

!SLIDE
# Append Only
# No B-trees!
# No locking!
# No big iron needed!

!SLIDE
# Filesystem!
# Implementable in Bash!
# Incremental backups!

!SLIDE
# No Erasers!
# BAD, BAD, BAD for financial records!

!SLIDE
# SQL or NoSQL

!SLIDE
# Wait there's more

!SLIDE
# Testing Made Easy/Simple
## Given a series of events has occurred
## When a single command is received
## Then a series of events occurs

!SLIDE
# Uses for Event Store
* building Domain Models
* building queary datastores
* building data warehouses (realtime!)
* monitoring - Oooh Pretty
* debugging

!SLIDE
## Uses for Event Store (cont.)
* enhance/replace model (benefit from learning)
* erase bugs/bad design
* build new read models
* testing
* persistence

!SLIDE
# time travel!

!SLIDE
# Events contain data points that may be used in ways yet to be known

!SLIDE
# Make marketing kiss your feet!

!SLIDE
# &lt;aside>

!SLIDE
# Dumb Concurrency Examples

!SLIDE
## computer science professors use strawmen

!SLIDE
    @@@ ruby
    attr_reader :balance
    def Account.withdraw(amount)
      @balance -= amount
    end

!SLIDE
FAIL!
    @@@ ruby
    account.balance.should == 20
    Thread.new { account.withdraw(10) }
    Thread.new { account.withdraw(10) }
    account.balance.should == 0

!SLIDE
    @@@ ruby
    def Account.withdraw(amount)
      # can never be too safe
      with_lock(Universe) do
        @balance -= amount
      end
    end

!SLIDE
## first year accounting students use event sourcing

!SLIDE
    @@@ruby
    def Account.debit(amount)
      @entries << Entry.new(:debit, amount)
    end
    def balance
      @entries.inject(@starting_balance) do
        |balance, entry|
        balance + entry.amount
      end
    end

!SLIDE
PASS!
    @@@ ruby
    account.balance.should == 20
    Thread.new { account.withdraw(10) }
    Thread.new { account.withdraw(10) }
    account.balance.should == 0

!SLIDE
# &lt;/aside>

!SLIDE
# In Review
* segregate commands from queries
* use many query optimized DBs
* sync models via events
* store events instead of domain objects
* build domain models from events

!SLIDE
## More info
## Greg Young
### CTO of IMIS
### stock market analytics firm
### sharing CQRS+ES 3+ years

!SLIDE
## Videos
### eXchange 2010: Greg Young on Architectural Innovation: Eventing, Event Sourcing
http://skillsmatter.com/podcast/design-architecture/architectural-innovation-eventing-event-sourcing
### QCon SF 08: Unshackle Your Domain
http://www.infoq.com/presentations/greg-young-unshackle-qcon08
### QCon London 2011: Events Are Not Just for Notifications
http://www.infoq.com/presentations/Events-Are-Not-Just-for-Notifications
### In The Brain of Greg Young: CQRS and Event Sourcing - the Business Perspective
http://skillsmatter.com/podcast/design-architecture/greg-young-cqrs-event-sourcing-the-business-perspective

!SLIDE
## MartinFowler
http://martinfowler.com/bliki/CQRS.html
http://martinfowler.com/eaaDev/EventSourcing.html
http://martinfowler.com/bliki/MemoryImage.html

!SLIDE
## Many frameworks on github
## Many examples as well

!SLIDE
# Q&A
