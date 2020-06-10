Live Project

Introduction

My last two weeks at the Tech Academy were spent as part of a development team. Our task was to create a new website for a local Portland theater using an ASP.NET MVC application in C#. Much of the new website had already been created by the time of my sprint, so there many front-end tasks to be accomplished along with the continuing back-end work. This allowed me the chance to exercise my skills in HTML, CSS, JavaScript, as well as C# and SQL. Beyond raw coding experience, these two weeks exposed me to the dynamics of team development within the context of DevOps and Scrum. I also gained more fluency in the use of Visual Studio, GitHub, and other tools. I know the future will provide me many opportunities to revisit these skills and expand on them.

Below are descriptions of the stories I worked on, as well as code snippets and screenshots to better illustrate the 

Archive Page Styling

The Archive Page, which displayed both current and past productions, had several display issued that needed to be addressed. I fixed the CSS which was improperly selecting the elements, I corrected the alternate text for each image, and I resized the Bootstrap pill badges which were large and unsightly.

Archive Page Responsiveness

Also on the Archive Page, the production photos were not responding to changes in window sizing. If there were too many in a row, they would be squeezed thin; if there were too few, they would stretch to occupy the entire viewport. The images were contained in a card-deck, so I adjusted the minimum width to ensure they would stack before shrinking too much. I also adjusted the maximum width of each to prevent the undue stretching. Unfortunately, when I changed the card styling it adversely affected the cards on all the other pages. So I created a custom class for the archive cards and applied the changes to that one.

.archivecard {
    min-width: 30rem;
    max-width: 40rem;
}



Easy Login Buttons

The website accomodates three types of users: Administrator, Cast Member, and Subscriber. There is a form each user must submit to be logged in properly, which requires inputs of name, user type, and password. I was tasked to create a workaround which would allow a user to bypass the login process with a click of a button. This was purely for the convenience of developers, and my code will be removed from the site before deployment.

<div class="col-md-4">
    <section class="loginpadding" id="loginForm">

      <h2 class="col-md-8 col-sm-8">Easy Login</h2>
      <p>This section exists for development purposes only. Please make sure that it is removed before deployment.</p> <!-- THE TEXT SAYS IT ALL -->
      <hr />
      <p><input type="button" class="iconBtn" value="Sign in as an Administrator" id="AdminBtn" /></p>
      <p><input type="button" class="iconBtn" value="Sign in as a Member" id="MemberBtn" /></p>
      <p><input type="button" class="iconBtn" value="Sign in as a Subscriber" id="SubscriberBtn" /></p>

    </section>
  </div>
  @*END Easy Login Buttons Section*@


  @*<div class="col-md-4">
        <section id="socialLoginForm">
            @Html.Partial("_ExternalLoginsListPartial", new ExternalLoginListViewModel { ReturnUrl = ViewBag.ReturnUrl })
        </section>
    </div>*@
</div>

@section Scripts {
  @Scripts.Render("~/bundles/jqueryval")
}

@*The JavaScript applies only to the Easy Login buttons, for the purposes of development. IT MUST BE REMOVED BEFORE DEPLOYMENT*@
<script type="text/javascript">
  document.getElementById("AdminBtn").addEventListener('click', function () {
    var email = document.getElementById('Email');
    email.value = 'test@gmail.com';
    var pass = document.getElementById('Password');
    pass.value = 'Passw0rd!';
    document.getElementById("login").submit();
  });

  document.getElementById("MemberBtn").addEventListener('click', function () {
    var email = document.getElementById('Email');
    email.value = 'member.test@gmail.com';
    var pass = document.getElementById('Password');
    pass.value = 'Ih@ve12cats';
    document.getElementById("login").submit();
  });

  document.getElementById("SubscriberBtn").addEventListener('click', function () {
    var email = document.getElementById('Email');
    email.value = 'subscriber.test@gmail.com';
    var pass = document.getElementById('Password');
    pass.value = '100100St!';
    document.getElementById("login").submit();
  });
</script>




CastMembers Image Update

Among the properties of the class CastMember was one called Photo. But the website also had entire class called Photo, and the designers wanted all images to be in the Photo class. My job was to replace the CastMember.Photo with a new property called PhotoId, which would store the ID of the Photo image. This story was completed in the following steps:

The image referenced by CastMember.Photo must be referenced by a new instance of Photo, but its id must match the proper CastMember. So I decided that when instantiating a new CastMember object, I would call the CreatePhoto method and create a new Photo whose Title was identical to the CastMember's Name property. Since CreatePhoto returns the Photo.PhotoId, I assigned that value to the new CastMember.PhotoId.

     public ActionResult Create()
        {
            ViewData["dbUsers"] = new SelectList(db.Users.ToList(), "Id", "UserName");
            
            return View();
        }

     
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "CastMemberID,Name,YearJoined,MainRole,Bio,PhotoId,CurrentMember,CastMemberPersonId,AssociateArtist,EnsembleMember,CastYearLeft,DebutYear")] CastMember castMember, HttpPostedFileBase file)
        {
            
            ModelState.Remove("CastMemberPersonID");

            
            string userId = Request.Form["dbUsers"].ToString();


            if (!string.IsNullOrEmpty(userId) && db.Users.Find(userId).CastMemberUserID != 0)
                ModelState.AddModelError("CastMemberPersonID", $"{db.Users.Find(userId).UserName} already has a cast member profile");

            if (ModelState.IsValid)
            {
                if (file != null && file.ContentLength > 0)
                {
                              
                    castMember.PhotoId = PhotoController.CreatePhoto(file, castMember.Name);
                }
               

            
                if (!string.IsNullOrEmpty(userId))
                {
                    castMember.CastMemberPersonID = db.Users.Find(userId).Id;
                }
               
                db.CastMembers.Add(castMember);
                db.SaveChanges();

               
                if (!string.IsNullOrEmpty(userId))
                {
                    
                    var selectedUser = db.Users.Find(userId);

          
                    selectedUser.CastMemberUserID = castMember.CastMemberID;

                  
                    db.Entry(selectedUser).State = EntityState.Modified;
                    db.SaveChanges();
                }

                return RedirectToAction("Index");
            }
            else  // This viewdata is required for the create view
            {
                ViewData["dbUsers"] = new SelectList(db.Users.ToList(), "Id", "UserName");
            }

            return View(castMember);
        }




All the CastMember CRUD pages now needed to be updated to accomodate the new PhotoId property. I called the DisplayPhoto method from the PhotoController on the relevant Index, Edit, Details, and Delete pages.

 <img class="castImage" src="@Url.Action("DisplayPhoto", "Photo", new { id = Model.PhotoId })" style="width:100%" />







I also altered the CastMember Create and Edit pages to display a preview of the image file which the user will assign to the cast member.

  <div class="form-group">
                @Html.Label("Photo")
              <div class="col-md-10 formBox">
                
                <img id="img" alt="" width="100" height="100" />
                <input type="file" name="file" class="fileSelect" onchange="document.getElementById('img').src = window.URL.createObjectURL(this.files[0])">
                @Html.ValidationMessageFor(model => model.PhotoId, "", new { @class = "text-danger" })

              </div>
            </div>




The website had been seeded with a dummy cast, and I had to adjust the seeding to use the new PhotoId. So I modified the Startup Page to seed all the new Photo images, then assign the CastMember.PhotoId the value of the id field in the new Photo database table.

    private void SeedCastPhotos()
        {
            var converter = new ImageConverter();
            // create images first
            string imagesRoot = Path.Combine(AppDomain.CurrentDomain.BaseDirectory + @"\Content\Images");

            Image image1 = Image.FromFile(Path.Combine(imagesRoot, @"London_Bauman.png"));
            Image image2 = Image.FromFile(Path.Combine(imagesRoot, @"JacQuelle_Davis.jpg"));
            Image image3 = Image.FromFile(Path.Combine(imagesRoot, @"Adriana_Gantzer.jpg"));
            Image image4 = Image.FromFile(Path.Combine(imagesRoot, @"Clara_Liis_Hillier.jpg"));
            Image image5 = Image.FromFile(Path.Combine(imagesRoot, @"Kaia_Maarja_Hillier.jpg"));
            Image image6 = Image.FromFile(Path.Combine(imagesRoot, @"Heath_Hyun_Houghton.jpg"));
            Image image7 = Image.FromFile(Path.Combine(imagesRoot, @"Tom_Mounsey.jpg"));
            Image image8 = Image.FromFile(Path.Combine(imagesRoot, @"Devon_Roberts.jpg"));

            var photos = new List<Photo>
            {
               new Photo
                {
                    OriginalHeight = image1.Height,
                    OriginalWidth = image1.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image1, typeof(byte[])),
                    Title = "London Bauman"
                },
                new Photo
                {
                    OriginalHeight = image2.Height,
                    OriginalWidth = image2.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image2, typeof(byte[])),
                    Title = "Jacquelle Davis"
                },
                new Photo
                {
                    OriginalHeight = image3.Height,
                    OriginalWidth = image3.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image3, typeof(byte[])),
                    Title = "Adriana Gantzer"
                },
                new Photo
                {
                    OriginalHeight = image4.Height,
                    OriginalWidth = image4.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image4, typeof(byte[])),
                    Title = "Clara-Liis Hillier"
                },
                new Photo
                {
                    OriginalHeight = image5.Height,
                    OriginalWidth = image5.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image5, typeof(byte[])),
                    Title = "Kaia Maarja Hillier"
                },
                new Photo
                {
                    OriginalHeight = image6.Height,
                    OriginalWidth = image6.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image6, typeof(byte[])),
                    Title = "Heath Hyun Houghton"
                },
                new Photo
                {
                    OriginalHeight = image7.Height,
                    OriginalWidth = image7.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image7, typeof(byte[])),
                    Title = "Tom Mounsey"
                },
                new Photo
                {
                    OriginalHeight = image8.Height,
                    OriginalWidth = image8.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image8, typeof(byte[])),
                    Title = "Devon Roberts"
                }
            };
            photos.ForEach(Photo => context.Photo.AddOrUpdate(p => p.PhotoFile, Photo));
            context.SaveChanges();
        }



        //Seeding database with dummy CastMembers
        private void SeedCastMembers()
        {
            //Add photos of cast members
            //string imagesRoot = Path.Combine(AppDomain.CurrentDomain.BaseDirectory + @"\Content\Images");

            var castMembers = new List<CastMember>
            {
                // For each cast member I replaced CastMember.Photo with CastMember.PhotoId and assigned it the value of the corresponding Photo.PhotoId 
                new CastMember{Name = "London Bauman", MainRole = Enum.PositionEnum.Actor,
                Bio = "London Bauman is an actor, writer, sound designer, and Theatre Vertigo company member. " +
                "As an artist, he is interested in immersive physical theatre, magical realism, and new collaborative works." +
                " Selected work includes the role of Torch in Beirut (The Steep & Thorny Way to Heaven), " +
                "Barnaby the Barkeep in the devised Western Melodrama Bang! (Action/Adventure’s pilot season) " +
                "and sound design / original compositions for The Romeo and Juliet Project (Enso Theatre Ensemble). " +
                "In August, London will be traveling to the Edinburgh Fringe Festival in Scotland as Robert in Chet Wilson’s new play Gayface.",
                PhotoId = context.Photo.Where(photo => photo.Title == "London Bauman").FirstOrDefault().PhotoId, 
                CurrentMember = true,},

                new CastMember{Name = "Jacquelle Davis", MainRole = Enum.PositionEnum.Actor,
                Bio = "Jacquelle Davis is a proud Portland native and member of Theatre Vertigo. " +
                "She studied acting at Willamette University. Jacquelle performs regularly with her beloved improv group, " +
                "No Filter. Her favorite roles include Jane Fonda in That Pretty Pretty; " +
                "Or, The Rape Play, and Box Worker 2 in Box. Jacquelle loves puns and pickles..",
                PhotoId = context.Photo.Where(photo => photo.Title == "Jacquelle Davis").FirstOrDefault().PhotoId,
                CurrentMember = true, },

                new CastMember{Name = "Adriana Gantzer", MainRole = Enum.PositionEnum.Actor,
                Bio = "Adriana has been a huge fan of Theatre Vertigo for many years and feels so fortunate to become " +
                "a part of this incredible company. She has been acting on stage for over a decade and has been a " +
                "full-time voiceover actor for over 4 years. Some favorite past roles include: Andy in A Dark Sky " +
                "Full of Stars, Adriana’s Theatre Vertigo debut; Matilde in The Clean House, Germaine in Picasso at " +
                "the Lapin Agile, and Georgeanne in Five Women Wearing the Same Dress. In her four years in Portland " +
                "she has worked with Milagro, NORTHWEST THEATRE WORKSHOP, Mask & Mirror, and Twilight theaters, " +
                "and at Prospect Theater Project in her hometown of Modesto, CA.",
                PhotoId = context.Photo.Where(photo => photo.Title == "Adriana Gantzer").FirstOrDefault().PhotoId,
                CurrentMember = true, },

                new CastMember{Name = "Clara-Liis Hillier", MainRole = Enum.PositionEnum.Actor,
                Bio = "Clara-Liis is a graduate of Reed College. A proud company member of Theatre Vertigo, " +
                "she is also a past resident actor for Bag&Baggage. She was last seen in Caucasian Chalk Circle " +
                "at Shaking the Tree, Gyspy at Broadway Rose, Godspell at Lakewood Theater " +
                "(Drammy Award for Supporting Actress), world premiere Carnivora as Woodwoman " +
                "(Theatre Vertigo) and as the Wicked Witch of the West in The Wizard of Oz (NW Children's Theater). " +
                "Favorite roles: Graeae Sister in Up The Fall with PHAME, Wait Until Dark (Susan Hendrix) with NWCTC, " +
                "Our Country's Good (Liz Morden) and Julius Caesar (Casca) with Bag&Baggage; The Seagull (Masha) with " +
                "NWCTC. When she's not onstage, Clara-Liis works for Portland Center Stage at The Armory as their" +
                " Education & Community Programs Associate and teaches Dance and Theater for NW Children's Theater " +
                "and Riverdale High School. Thank you to Heath K. for his love and patience and Mom and " +
                "Kaia for their strength and inspiration. For Ted.",
                PhotoId = context.Photo.Where(photo => photo.Title == "Clara-Liis Hillier").FirstOrDefault().PhotoId,
                CurrentMember = true, },

                new CastMember{Name = "Kaia Maarja Hillier", MainRole = Enum.PositionEnum.Actor,
                Bio = "Kaia is a current Theatre Vertigo company member, a Board and Company member of the Pulp Stage, " +
                "a No Filter Improv Troupe member, and Costume Designer. Past acting credits include: Jessica in A Maze" +
                " (Theatre Vertigo); Nora in Assistance (Theatre Vertigo); and April in The Best of Everything (Bag &Baggage)." +
                " Kaia has had SO much fun performing with the new ensemble in readings that celebrate Vertigo's " +
                "rich artistic past. Thank you to everyone who has come out to support the new ensemble and to " +
                "helping keep Theatre Vertigo and the Shoebox thriving-we need these space to stay alive and " +
                "let our community grow and share their art. Much love to Mom, the Ensemble, the Associate Artists, " +
                "Clara, and JQ.",
                PhotoId = context.Photo.Where(photo => photo.Title == "Kaia Maarja Hillier").FirstOrDefault().PhotoId,
                CurrentMember = true, },

                new CastMember{Name = "Heath Hyun Houghton", MainRole = Enum.PositionEnum.Actor,
                Bio = "A Korean American actor, writer and director.  He previously appeared with Theatre Vertigo in Assistance;" +
                " other Portland credits include work with Imago Theatre, Portland Shakespeare Project, Broadway Rose Theatre" +
                ", and many more.  Exploring the relationships between the sciences and the arts is a focal point of his work" +
                " as a collaborator and educator.",
                PhotoId = context.Photo.Where(photo => photo.Title == "Heath Hyun Houghton").FirstOrDefault().PhotoId,
                CurrentMember = true, },

                new CastMember{Name = "Tom Mounsey", YearJoined= 2012, MainRole = Enum.PositionEnum.Actor,
                Bio = "Tom found a passion for theatre and performance in his late 20s thanks to a class at Portland" +
                " Actors Conservatory, and has been acting in and around Portland since his graduation in 2008. " +
                "You might have seen him on stage at places like defunkt theatre, Imago Theatre, Northwest Classical " +
                "Theatre Collaborative, Action/Adventure Theatre, Lakewood Center for the Arts, Clackamas Repertory " +
                "Theatre, and of course, Theatre Vertigo. Tom was a member of Theatre Vertigo from 2012 to 2017, " +
                "and is very excited to be back as part of this amazing company.",
                PhotoId = context.Photo.Where(photo => photo.Title == "Tom Mounsey").FirstOrDefault().PhotoId,
                CurrentMember = true, },

                new CastMember{Name = "Devon Roberts", MainRole = Enum.PositionEnum.Actor,
                Bio = "Devon Roberts is a born and raised Portland director and actor. He holds a BA of Theater Arts " +
                "from Portland State University and is an alumnus of the Orchard Project Core Company. He has worked" +
                " with local companies: Boom Arts, Fuse Theatre Ensemble, Portland Center Stage at The Armory and out" +
                " of state: such as The Civilians, Tectonic Theater Project, Pig Iron and at the Edinburgh Fringe Festival." +
                " When Devon isn’t working on and off stage, he can be found enjoying the local cuisine, or soaking up" +
                " the natural beauty of Oregon. Devon is thankful for the opportunity to join the Vertigo Ensemble!",
                PhotoId = context.Photo.Where(photo => photo.Title == "Devon Roberts").FirstOrDefault().PhotoId,
                CurrentMember = true, },

                };

            castMembers.ForEach(castMember => context.CastMembers.AddOrUpdate(c => c.Name, castMember));
            context.SaveChanges();
        }


