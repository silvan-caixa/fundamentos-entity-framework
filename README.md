# fundamentos-entity-framework
curso básico banco de dados - orm

## Mapeamento Anotações



## Mapeamento Fluente

PROJETO MAP Fluente
DATA
    => BlogDataContext.cs
        BlogDataContext : DbContext
        {
            DebSet<User> Users, DebSet<Post> Posts, DebSet<Category> Categories  

            OnConfiguring(DbContextOptionBuider options)
            => options.UserSqlerver("Server=Localhost,1433;Database=BancoDados; User ID=Usuario;Password=123456")

            void OnModelCreating(ModelBuilder modelBuilder)
            {
                modelBuilder.ApplyConfiguration(new CategoryMap())
                modelBuilder.ApplyConfiguration(new UserMap())
                modelBuilder.ApplyConfiguration(new PostMap())
            }
        }
    Mappings
        => CategoryMap.cs
            CategoryMap : IEntityTypeConfiguration<Category>
            {
                void Configure(EntityTypeBuilder<Category> builder)
                {
                    //TABELA
                    Builder.ToTable("Category");

                    //CHAVE PRIMARIA
                    Builder.HasKey(x=>x.Id);
                    
                    //IDENTITY
                    Builder.Property(x=>x.Id)
                    .ValueGeneratedOnAdd()
                    .UserIdentityColumn()
                    
                    //PROPRIEDADES
                    Builder.Property(x=>x.Name)
                    .IsRequired()
                    .HasColumnName(Name)
                    .HasColumnType(Nvarchar)
                    .HasMaxLength(80)

                    //INDEX
                    Builder.HasIndex(x=>x.Slug, "IX_Category_Slug")
                    .IsUnique()
                }
            }
        => UserMap.cs
            UserMap : IEntityTypeConfiguration<User>
            {
                void Configure(EntityTypeBuilder<User> builder)
                {
                    ...
                    //RELACIOMANTO muitos users pra muitos roles
                    builder.HasMany(x => x.Roles)
                    .WithMany(x => x.Users)
                    .UsingEntity<Dictionary<string, object>>(
                        "UserRole",
                            role => role
                            .HasOne<Role>()
                            .WithMany()
                            .HasForeingKey("RoleId")
                            .HasContraintName("FK_UserRole_RoleId")
                            .HasDelete(DeleteBehavior.Cascade),

                            user => user
                            .HasOne<User>()
                            .WithMany()
                            .HasForeingKey("UserId")
                            .HasContraintName("FK_UserRole_UserId")
                            .HasDelete(DEleteBehavior.Cascade)
                    )
                }
            }
        => PostMap.cs
            UserMap : IEntityTypeConfiguration<Post>
            {
                void Configure(EntityTypeBuilder<Post> builder)
                {
                    ...

                    //PARA DATAS
                    builder.Proprty(x=x.LastUpdateDate)
                    .IsRequired()
                    .HasColumn("LastUpdateDate")
                    .HasType("SMALLDATETIME")
                    .HasDefaultValueSql("GETDATE()") // Quando o banco gera a data default
                    .HasDefaultValue("DateTime.Now.ToUniveralTime()") / Quando a data default é gerada no codigo 

                    //RELACIONAMENTO 1 AUTHOR PARA MUITOS Posts
                    builder.HasOne(x => x.Author)
                    .WithMany(x => x.Posts)
                    .HasContraintName("FK_Post_User")
                    .OnDelete("DeleteBehavior.Cascade");

                    builder.HasOne(x => x.Category)
                    .WithMany(x => x.Posts)
                    .HasContraintName("FK_Post_Category")
                    .HasDelete(DeleteBehavior.Cascade);

                    //RELACIOMANTO  MUITOS POSTS PARA MUITOS TAGS

                    builder.HasMany(x => x.Tags)
                    .WithMany(x => x.Posts)
                    .UsingEntity<Dictionary<string, object>>(
                        "PostTag",
                        post => post.HasOne<Tag>()
                            .WithMany()
                            .HasForeingKey("PostId")
                            .HasContraintName("FK_PostTag_PostId")
                            .OnDelete(DeleteBehavior.Cascade),
                        tag => tag.HasOne<Post>()
                            .WithMany()
                            .HasForeingKey("TagId")
                            .HasContraintName("FK_PostTag_TagId")
                            .OnDelete(DeleteBehavior.Cascade)
                            
                    )
                    

                }
            }

//primeira etapa
MODELS
    => User.cs
        Id,Name,Email,PasswordHash,Image,Slug,Bio
        IList<Post>Posts, IList<Role>Roles
    => Category.cs
        Id,Name,Slug
        IList<Post>Posts
    => Post.cs
        Id,Title,Sumary,Body,Slug,CreateDate,LastUpdateDate,Category, Author
        IList<Tag>Tags
    => Tag.cs
        Id,Name,Slug
        IList<Post>Posts
    => Role.cs
        Id, Name, Slug
        IList<User>User


//ultima etapa
Main
    using var context = new BlogDataContext();

    var user = context.Users.FirstOrDefault();

    context.Users.Add(new User{
        Bio = """
        Email = """
        Image = ""
        Name = ""
        PasswordHash = ""
        Slug = ""

    });

    var post = new Post{
        Author = user,
        Body = "",
        Category = new Category{
            Name = "BackEnd",
            Slug = "backend"
        },
        CreateDate = System.DateTime.Now,
        Slug = """
        Summary = ""
        Title = """

    }
    context.Posts.Add(post);
    context.SaveChanges();
