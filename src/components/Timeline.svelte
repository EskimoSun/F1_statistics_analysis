<script>
    import * as d3 from "d3";
    import { onMount } from "svelte";
</script>

<div class="timeline">
    <div class="container right">
        <div class="content">
            <h2 class='schumacher'>The Beginning: A Star is Born (1991)</h2>
            <p>
                Imagine stepping into the world's most elite racing arena with just a day's notice. That's exactly what Michael Schumacher did, making his Formula 1 debut with Jordan Racing at the Belgian Grand Prix in 1991. Despite having minimal experience, he qualified 7th, outpacing his veteran teammate. A fairy-tale start to what would be a legendary career.
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='schumacher'>First Taste of Victory: Belgium, Again! (1992)</h2>
            <p>
                Just a year after his debut, Schumacher found himself atop the podium for the first time at the Belgian Grand Prix, proving that his initial success was no fluke. This victory wasn't just a win; it was a declaration that a new titan had arrived in Formula 1.
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='schumacher'>Rising Through the Ranks: The Benetton Years (1994-1995)</h2>
            <p>
                Fast-forward to 1994, and imagine the tension, the drama as Schumacher clinches his first World Championship. The following year, he does it again, solidifying his status not as a one-hit wonder but a true powerhouse in the racing world.
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='schumacher'>The Ferrari Saga Begins: Building a Dynasty (1996-2000)</h2>
            <p>
                Schumacher's move to Ferrari in 1996 was like joining a band on the brink of breakup and turning them into rock legends. The team hadn't won a championship in 16 years. Schumacher's quest with Ferrari was tough, filled with near misses but also moments of brilliance, laying the groundwork for what was to come.
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='schumacher'>Domination Era: Rewriting History (2000-2004)</h2>
            <p>
                Now, imagine being so dominant that victory seems a foregone conclusion. From 2000 to 2004, Schumacher and Ferrari were invincible, securing five consecutive World Championships. This era wasn't just about winning; it was about redefining excellence, with Schumacher setting records that seemed etched in stone.
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='hamilton'>A Dream Begins: The First Chapter (2007)</h2>
            <p>
                Imagine stepping into the high-octane world of Formula 1 and immediately turning heads. In 2007, a young Lewis Hamilton did just that, making his debut with McLaren. From the get-go, he was not just competing; he was challenging for the title, finishing his first season with an astonishing 4 wins and narrowly missing the championship by a single point. A star was born.        
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='hamilton'>Breaking Barriers: A Champion Crowned (2008)</h2>
            <p>
                Just one year later, in a season filled with drama and heart-stopping moments, Hamilton clinched his first World Championship, becoming the youngest driver to do so at the time. His victory at the Brazilian Grand Prix, won by a mere point in the final corners of the last lap, was not just a win but a statement to the racing world.        
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='schumacher'>The Comeback: A Legend Returns (2010-2012)</h2>
            <p>
                Retirement couldn't hold Schumacher back. In 2010, he returned with Mercedes, not just to race, but to inspire a new generation. Though he didn't add to his championship tally, his return was a testament to his enduring passion and influence in the sport.            
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='schumacher'>Legacy Cemented: A Farewell to Arms (2012)</h2>
            <p>
                Schumacher's final race in 2012 marked the end of an era. With 91 Grand Prix wins, 7 World Championships, and countless records, his career was a blueprint in mastery, resilience, and the relentless pursuit of greatness.            
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='hamilton'>A New Era: Joining Mercedes (2013)</h2>
            <p>
                In a move that surprised many, Hamilton left McLaren to join Mercedes in 2013. Critics questioned the decision, but Hamilton saw it as a challenge. This wasn't just a change of teams; it was a leap of faith into building a legacy.    
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='hamilton'>Dominance Redefined: Shattering Records (2014-2020)</h2>
            <p>
                Hamilton and Mercedes became synonymous with victory, embarking on a journey that would redefine the sport. Winning 6 World Championships in 7 years, Hamilton’s era was not merely about triumphs but about pushing the limits of what was possible, breaking records for most wins, pole positions, and podium finishes along the way.    
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='hamilton'>Champion for Change: Racing for More than Wins (2020-Present)</h2>
            <p>
                Hamilton's influence stretches beyond the track. He's used his platform to advocate for diversity, equality, and environmental sustainability within and outside F1. Winning his 7th World Championship in 2020, he matched Schumacher's record, but his ambitions were larger: to drive change in the sport he loves.    
            </p>
        </div>
    </div>
    <div class="container right">
        <div class="content">
            <h2 class='hamilton'>Perseverance and Passion: The Quest Continues (Present)</h2>
            <p>
                Each season, Hamilton continues to compete at the pinnacle of his powers, not just racing for records but for a cause, for change, and for the sheer love of the sport. His story is ongoing, a narrative of resilience, dedication, and the relentless pursuit of greatness.    
            </p>
        </div>
    </div>
</div>

<style>
    .timeline {
        position: relative;
        max-width: 1200px;
        margin-left: 10%;
        margin-top: 20px;
        overflow-y: scroll;
        overflow-x: hidden;
        height:70vh;
        
    }

    .timeline::after {
        content: "";
        position: absolute;
        width: 6px;
        background-color: #FAEF5D;
        top: 0;
        bottom: 0;
        height: 2100px;
        left: 10%;
        margin-left: -3px;
    }

    .container {
        padding: 10px 40px;
        position: relative;
        background-color: inherit;
        width: 70%;
    }

    .container::after {
        content: "";
        position: absolute;
        width: 25px;
        height: 25px;
        right: -17px;
        background-color: white;
        border: 4px solid #76ABAE;
        top: 15px;
        border-radius: 50%;
        z-index: 1;
    }

    .right {
        left: 10%;
    }

    .right::before {
        content: " ";
        height: 0;
        position: absolute;
        top: 22px;
        width: 0;
        z-index: 1;
        left: 30px;
        border: medium solid white;
        border-width: 10px 10px 10px 0;
        border-color: transparent white transparent transparent;
    }

    .right::after {
        left: -16px;
    }

    .content {
        padding: 1px 30px;
        background-color: white;
        position: relative;
        border-radius: 6px;
        color: black;
    }

    .schumacher{
        color: #EF1A2D;
    }

    .hamilton{
        color: #00A19B;
    }
</style>
