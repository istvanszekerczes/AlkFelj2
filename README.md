# AlkFelj2

Console app -> Debug Properties CSV path -> CSVReader nuget -> CSV beolvasas -> Model osztaly -> SafeInt32Converter -> LINQ feladatok
https://www.tutorialsteacher.com/linq  -> LINQ tutorial

Read in csv:
private static IEnumerable<T> LoadData<T>(string path) {
    if (!File.Exists(path)) {
        Console.WriteLine($"Hiba. A megadott fájl útvinala nem létezik: {path}");
        Enviroment.Exit(1);
    }

    var configuration = new CsvConfiguration(System.Globalization.CultureInfo.InvariantCulture)
    {
        Encoding = System.Text.Encoding.UTF8,
        Delimiter = ",",
        BadDataFound = (args) => { },
        MissingFieldFound = (args) => { },
    }

    
    using var reader = new StreamReader(path);
    using var csv = new CsvReader(reader, config);
    return csv.GetRecords<T>().ToList();
}

Join:
var athletes = new List<Athlete>();
var teams = new List<Team>();

var result =
    from a in athletes
    join t in teams
    on a.TeamId equals t.Id
    select new { AthleteName = a.Name, TeamName = t.Name };

##Model:
namespace ConsoleApp1.models
{
    public class SafeInt32Converter : Int32Converter 
    {
        public override object? ConvertFromString(string? text, IReaderRow row, MemberMapData MemberMapData) 
        {
            if (int.TryParse(text, out int variable)) 
            {
                return variable;
            }
            return 0;
        }
    }

    internal class Athlete
    {
        //ID,Name,Sex,Age,Height,Weight,Team,NOC

        public int ID { get; set; }
        public  String? Name { get; set; }
        public String? Sex { get; set; }
        public int? Age { get; set; }
        [TypeConverter(typeof(SafeInt32Converter))]
        public int? Height { get; set; }
        public int? Weight { get; set; }
        public String? Team { get; set; }
        public String? NOC { get; set; }


    }
}

Converter
using System;
    using System.Collections.Generic;
    using System.ComponentModel;
    using System.Globalization;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    
    namespace ConsoleApp1.models
    {
    
        public class SafeInt32Converter : Int32Converter
        {
            public override object? ConvertFrom(ITypeDescriptorContext? context, CultureInfo? culture, object value)
            {
                if (value is string s)
                {
                    if (int.TryParse(s, NumberStyles.Integer, culture ?? CultureInfo.InvariantCulture, out int result))
                        return result;
                    return 0;
                }
    
                return base.ConvertFrom(context, culture, value);
            }
    
            public override object? ConvertTo(ITypeDescriptorContext? context, CultureInfo? culture, object? value, Type destinationType)
            {
                if (destinationType == typeof(string) && value is int i)
                    return i.ToString(culture ?? CultureInfo.InvariantCulture);
    
                return base.ConvertTo(context, culture, value, destinationType);
            }
        }
    
    
        internal class Athlete
        {
            //ID,Name,Sex,Age,Height,Weight,Team,NOC
    
            public int ID { get; set; }
            public  string? Name { get; set; }
            public string? Sex { get; set; }
            public int? Age { get; set; }
    
            [TypeConverter(typeof(SafeInt32Converter))]
            public int? Height { get; set; }
            public int? Weight { get; set; }
            public string? Team { get; set; }
            public string? NOC { get; set; }
    
    
        }
    }

Full Code:
using System.Xml.Linq;
using ConsoleApp1.models;
using CsvHelper;
using CsvHelper.Configuration;

namespace ConsoleApp1
{
    internal class Program
    {
        static void Main(string[] args)
        {

            Console.WriteLine(args[0]);

            if (!File.Exists(args[0]))
            {
                Console.WriteLine("Wrong file path");
                Environment.Exit(1);
            }
            if (args.Length != 1)
            {
                Console.WriteLine("Wrong number of args");
                Environment.Exit(1);
            }

            var config = new CsvConfiguration(System.Globalization.CultureInfo.InvariantCulture)
            {
                Encoding = System.Text.Encoding.UTF8,
                Delimiter = ",",
                BadDataFound = (args) => { },
                MissingFieldFound = (args) => { },
            };

            using var reader = new StreamReader(args[0]);
            using var csv = new CsvReader(reader, config);
             
            var athletes = csv.GetRecords<Athlete>().ToList();

            // 3. LINQ segítségével határozd meg az összes sportoló átlagéletkorát.

            Console.WriteLine("3:");
            Console.WriteLine($"{athletes.Average(x => x.Age):N2}" + "\n");

            // 4. LINQ segítségével listázd ki a legmagasabb sportoló nevét és magasságát.
            Console.WriteLine("4:");
            int maxHeight = athletes.Max(x => x.Height);
            Athlete tallest = athletes.Where(x => x.Height == maxHeight).First();
            Console.WriteLine($"{tallest.Name}: {tallest.Height}");

            // 5. LINQ segítségével számold meg, hogy csapatonként hány sportoló szerepel az adathalmazban.
            Console.WriteLine("\n5:");
            var groupByTeam = athletes.GroupBy(x => x.Team).OrderByDescending(x => x.Count());
            foreach (var group in groupByTeam)
            {
                Console.WriteLine($"{group.Key}");
                Console.WriteLine($"{group.Count()}");
            }

            //6. LINQ segítségével atározd meg a női sportolók átlagmagasságát.

            Console.WriteLine("\n6:");

            Console.WriteLine(athletes.Where(x => x.Sex == "F").Average(x => x.Height));

            //7. LINQ segítségével listázd ki azoknak a sportolóknak a nevét ABC sorrendben, akik 25 év alattiak
            Console.WriteLine("\n7:");

            var youngs = athletes.Where(x => x.Age < 25).OrderBy(x => x.Name);

            foreach (var y in youngs)
            {
                Console.WriteLine(y.Name);
            }

            //8. LINQ segítségével minden csapatból keresd meg a legnehezebb sportolót (név és testsúly)
            Console.WriteLine("\n8:");

            foreach (var group in groupByTeam)
            {
                Console.WriteLine($"{group.Key} : ");
                Console.WriteLine(group.OrderByDescending(x => x.Weight).First().Name);
                Console.WriteLine(group.OrderByDescending(x => x.Weight).First().Weight + "\n");
            }

            //9. LINQ segítségével csoportosítsd a sportolókat nem szerint, és számítsd ki mindkét csoport átlagéletkorát.
            Console.WriteLine("\n9:");

            var groupBySex = athletes.GroupBy(x => x.Sex).OrderByDescending(x => x.Count());

            foreach (var group in groupBySex) 
            {
                Console.WriteLine(group.Key + ": ");
                Console.WriteLine(group.Average(x => x.Age));
            }
        }
    }
}

