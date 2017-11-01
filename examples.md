# orb.v2.js examples

Please note: There is no compatibility v1 and v2.

## Files
**orb.v2.js**:
all-in-one package

**orb-core.v2.js**:
Truncated version of orb.js. Core functions for orb-satellite.v2.js or orb-planetary.v2.js.

**orb-satellite.v2.js**:
Truncated version of orb.js. If you only need calculations for artificial satellites or space debris, you use this .js file with orb-core.v2.js.

**orb-planetary.v2.js**:
Truncated version of orb.js. If you only need calculations for planetary objects such as planets or asteroids, you use this .js file with orb-core.v2.js.

## orb-core.v2.js

### Orb
Returns information of version, author, license

    {
      "VERSION":<string>,
      "AUTHOR":<string>,
      "LICENSE":<string>
    }

### Orb.Const
Various numerical constants for astronomical calculations

    }
      "PI","RAD","AU","RE","LD","LY","PC","PARSEC","G": <number>,
      "Planets":<array>, //list of planets,
      "Sun","Mercury","Venus","Earth","Moon","Mars","Jupiter","Saturn","Uranus","Neptune":{
      "radius":<number>, //radius at equator,
      "obliquity":<number>, //obliquity of the planet,
      "mass":<number>, //mass of the planet,
      "gm":<number> //gravitational constant for the planet
    }

### Orb.Time()
Convert Date to the astronomical time units

    //initialize
    var date = new Date();
    var time = new Orb.Time(date);
    //if you omit date object, you get value of now.

    var gmst = time.gmst();
    var julian_date = time.jd();
    var time_in_day = time.time_in_day();
    var day_of_year = time.doy();

## Orb.Observation()
Calc horizontal coordinates of target from observer.

    //initialize
    var date = new Date()
    var your_location = {
      "latitude":35.658,
      "longitude":139.741,
      "altitude":0
    }

    // for planets (see Orb.VSOP)
    var mars = new Orb.VSOP("Mars");
    var obs = new Orb.Observation({"observer":your_location,"target":mars});
    var azel = obs.azel(date); // azimuth, elevation -> horizontal coordinates

    //for artificial satellites (see Orb.SGP4)
    var tle = {
      first_line:"1 25544U 98067A   15012.59173611  .00014829  00000-0  23845-3 0  7179",
      second_line:"2 25544 051.6466 140.7335 0006107 243.2909 291.5211 15.53213268923827"
    }
    var satellite = new Orb.SGP4(tle);
    var obs = new Orb.Observation({"observer":your_location,"target":satellite});
    var azel = obs.azel(date); // azimuth, elevation -> horizontal coordinates

    //for equatorial sphirical coordinates (RA,Dec)
    var obs = new Orb.Observation({
      "observer":your_location ,
      "target":{"ra":0,"dec":0}
    });
    var azel = obs.azel(date); // azimuth, elevation

    //for equatorial rectangular coordinates (x,y,z)
    var obs = new Orb.Observation({
      "observer":your_location ,
      "target":{"x":0,"y":0,"z":0}
    });

    var azel = obs.azel(date); // azimuth, elevation

    //if target includes "coorinate_keywords" with "ecliptic", orb.js automatically convert coordinates.
    // "coorinate_keywords":"ecliptic rectangular"

        var obs = new Orb.Observation({
          "observer":your_location ,
          "target":{
            "x":0,
            "y":0,
            "z":0,
            "date":new Date(),
            "coorinate_keywords":"ecliptic rectangular"
          }
        });
        //if you omit date object, you get value of the time of observation.

        var azel = obs.azel(date); // azimuth, elevation

### Orb.RadecToXYZ()
Convert equatorial spherical coordinates(RA, Dec) to equatorial rectangular coordinates(x,y,z)
Please note: "ra"(right ascension) must be hours.

    var sirius = {
      "ra":6.75257,
      "dec":-16.7131,
      "distance":543300
    }
    var xyz = Orb.RadecToXYZ(sirius)

### Orb.XYZtoRadec()
Convert equatorial rectangular coordinates(x,y,z) to equatorial spherical coordinates(RA, Dec)
Please note: "ra"(right ascension) must be hours.

    var sirius = {
      "x": -101858.45670898016,
      "y": 510282.46985869075,
      "z": -156241.94619812947
    }

    var radec = Orb.XYZtoRadec(sirius)

if "coorinate_keywords" includes "ecliptic" (and optional "date"), this method automatically convert ecliptic coordinate to equatorial coordinate.

    var sirius = {
      "x": -101858.45670898016,
      "y": 510282.46985869075,
      "z": -156241.94619812947,
      "date":new Date(),
      "coorinate_keywords":"ecliptic rectangular"
    }
    //if you omit date object, you get value of now.

    var radec = Orb.XYZtoRadec(sirius)

### Orb.EquatorialToEcliptic()
Convert equatorial rectangular coordinates(x,y,z) to ecliptic rectangular coordinates(x,y,z)

    var sirius = {
      "x": -101858.45670898016,
      "y": 510282.46985869075,
      "z": -156241.94619812947
    }
    var ecliptic = Orb.EquatorialToEcliptic({
      "equatorial": sirius,
      "date":date
    })


### Orb.EclipticToEquatorial()
Convert ecliptic rectangular coordinates(x,y,z) to equatorial rectangular coordinates(x,y,z)

     var sirius = {
       "x":-101858.45670898016,
       "y":510282.46985869075,
       "z":-156241.94619812947
     }
    var equatorial = Orb.EclipticToEquatorial({
      "ecliptic": sirius,
      "date":date
    })


### Orb.NutationAndObliquity()
Calc nutation and obliquity of Earth

    var date = new Date()
    var nao = Orb.NutationAndObliquity(date)
    var obliquity = nao.obliquity()
    var nutation = nao.nutation()

### Orb.Obliquity()
Calc obliquity of Earth. This is wrapper function of Orb.NutationAndObliquity()

    var date = new Date()
    var obliquity = Orb.Obliquity(date)


### Orb.RoundAngle()

    var degree = 400;
    var round = Orb.RoundAngle(degree) // returns 40

    var degree = -10;
    var round = Orb.RoundAngle(degree) // returns 350


## orb-satellite.v2.js (require orb-core.v2.js)

### Orb.SGP4()

    //initialize
    var tle = {
      first_line:"1 25544U 98067A   15012.59173611  .00014829  00000-0  23845-3 0  7179",
      second_line:"2 25544 051.6466 140.7335 0006107 243.2909 291.5211 15.53213268923827"
    }
    var satellite = new Orb.SGP4(tle);

    var xyz = satellite.xyz(date); // equatorial rectangular coordinates (Earth centered): x, y, z
    var latlng = satellite.latlng(date); // geographic spherical coordinates :  latitude, longitude, altitude

### Orb.DecodeTLE()

## orb-planetary.v2.js (require orb-core.v2.js)
This is truncated version of orb.js. If you only need calculations for planetary objects such as planets, asteroids and so on, you use this .js file with orb-core.v2.js.

### Orb.kepler()

    //initialize
    var asteroid = new Orb.Kepler({
      "gm": 2.9591220828559093E-4; //in au^3/d^2
      "eccentricity":"0.08728849329001058",
      "inclination":"6.812676631845272", //in degree
      "longitude_of_ascending_node":"250.5660658100269", //in degree
      "argument_of_periapsis":"95.63473165761138", //in degree
      "time_of_periapsis":"2456918.756066796", //in jd
      "semi_major_axis":"1.001911878091084" //in au
    });

    var xyz = asteroid.xyz(date); // x, y, z -> ecliptic rectangular coordinates in au
    var radec = asteroid.radec(date); // ra, dec, distance -> equatorial spherical coordinates, distance in au

"gm" has default value "2.9591220828559093E-4" so you can omit "gm" for the Sun.


### Orb.VSOP()
  ["Mercury","Venus","Earth","Moon","Mars","Jupiter","Saturn","Uranus","Neptune"]

    //initialize
    var mars = new Orb.VSOP("Mars");
    // var mars = new Orb.Mars();
    // this is same as above

    var xyz = mars.xyz(date); // x, y, z -> ecliptic rectangular coordinates
    var radec = mars.radec(date); // ra, dec, distance -> equatorial spherical coordinates

### Orb.Luna()
Position and Phase of the moon seen from the Earth.
Please note: xyz() returns Earth centered "equatorial" rectangular coordinates not "ecliptic"

    //initialize
    var luna = new Orb.Luna();
    // var luna = new Orb.Moon();
    // this is same as above

    var xyz = luna.xyz(date); // x, y, z -> equatorial rectangular coordinates (Earth centered)
    var radec = luna.radec(date); // ra, dec, distance -> equatorial spherical coordinates

### Orb.Sun()
position of the sun seen from the Earth.
Please note: xyz() returns Earth centered "equatorial" rectangular coordinates not "ecliptic"

    //initialize
    var sun = new Orb.Sun();

    var xyz = sun.xyz(date); // x, y, z -> equatorial rectangular coordinates (Earth centered)
    var radec = sun.radec(date); // ra, dec, distance -> equatorial spherical coordinates


# sapplemental

## orb-data-handler.js
### Orb.DataLoader()

    var data = Orb.DataLoader({
      format:"text", // or "json" or "xml"
      path: <path_to_data>,
      id:"id",
      ajax:true, // or "false"
      callback:function(data,id){
      // do something
      }
    })

## orb-date-handler.js
### Orb.DigitsToDate()

    var date = Orb.DigitsToDate("20170101120000"); // return date object 2017.01.01 12:00:00 UTC

### Orb.DateToDigits()

    var date = new Date();
    date.setTime(Date.UTC(2017,0,1,12,0,0))    
    var digits = Orb.DateToDigits(date); //return "20170101120000"

### Orb.StringToDate()

    //var str = "2017-01-01T12:00:00"    
    //date separator should be "." or "-", time and date separator should be " " or "T" and time separator should be ":"

    var date = Orb.StringToDate(str)

### Orb.FormatUTCDate()

    var date = new Date();
    date.setTime(Date.UTC(2017,0,1,12,0,0))  
    var str = Orb.FormatUTCDate(date) // return "2017.01.01 12:00:00"

### Orb.FormatLocalDate()

    var date = new Date();
    date.setTime(Date.UTC(2017,0,1,12,0,0))  
    var str = Orb.FormatUTCDate(date) // return "2017.01.01 21:00:00" in timezone GMT+9:00